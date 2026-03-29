### Windows Remote Management Protocols

Windows systems utilize several protocols for local and remote administration. These services are often enabled by default on modern Windows Server versions to facilitate centralized management.

---

### Remote Desktop Protocol (RDP)

**RDP** provides a graphical interface for remote computer access, typically operating over **TCP/UDP port 3389**.

#### 1. Footprinting the Service

Scanning RDP reveals critical information such as **Network Level Authentication (NLA)** status, product versions, and hostnames.

|Tool|Command|Purpose|
|:--|:--|:--|
|**Nmap**|`nmap -sV -sC <TARGET_IP> -p3389 --script rdp*`|Enumerates encryption settings, NLA status, and NetBIOS/DNS info.|
|**Nmap**|`nmap -sV -sC <TARGET_IP> -p3389 --packet-trace --disable-arp-ping -n`|Tracks individual packets for manual inspection of RDP cookies.|

**Operational Note:** Threat hunters and **EDR** systems can identify Nmap's default RDP cookie (`mstshash=nmap`), potentially leading to a lockout on hardened networks.

#### 2. Identifying Security Settings

The `rdp-sec-check.pl` script unauthentically identifies supported protocols and encryption methods.

**Workflow: Installing and Running RDP Security Check**

1. Install the required Perl module:
    
    ```
    sudo cpan install Encoding::BER
    ```
    
2. Clone the repository:
    
    ```
    git clone https://github.com/CiscoCXSecurity/rdp-sec-check.git && cd rdp-sec-check
    ```
    
3. Execute the check against the target:
    
    ```
    ./rdp-sec-check.pl <TARGET_IP>
    ```
    

#### 3. Initiating a Session

Once credentials or vulnerabilities are identified, Linux-based attackers can interact with the GUI using tools like `xfreerdp`.

|Tool|Command|
|:--|:--|
|**xfreerdp**|`xfreerdp /u:<USERNAME> /p:"<PASSWORD>" /v:<TARGET_IP>`|

---

### Windows Remote Management (WinRM)

**WinRM** is a command-line management protocol based on **SOAP**. It is required for PowerShell remoting and event log merging.

#### 1. Service Identification

WinRM typically uses **TCP port 5985 (HTTP)** or **5986 (HTTPS)**.

```
nmap -sV -sC <TARGET_IP> -p5985,5986 --disable-arp-ping -n
```

#### 2. Establishing a Remote Shell

**WinRS** (Windows Remote Shell) or **Evil-WinRM** (on Linux) can be used to execute arbitrary commands on the remote system.

|Tool|Command|Purpose|
|:--|:--|:--|
|**PowerShell**|`Test-WsMan -ComputerName <TARGET_IP>`|Verifies if the remote server is reachable via WinRM.|
|**Evil-WinRM**|`evil-winrm -i <TARGET_IP> -u <USERNAME> -p <PASSWORD>`|Provides an interactive PowerShell shell on the target.|

---

### Windows Management Instrumentation (WMI)

**WMI** is the most critical interface for Windows administration, allowing read/write access to almost all system settings.

#### Footprinting and Execution

WMI communication initializes on **TCP port 135** before moving to a random port for data transfer.

|Tool|Command|Purpose|
|:--|:--|:--|
|**Impacket**|`/usr/share/doc/python3-impacket/examples/wmiexec.py <USERNAME>:"<PASSWORD>"@<TARGET_IP> "hostname"`|Executes a command on the target via WMI.|

---

### Dangerous Settings and Misconfigurations

| Feature/Setting              | Attack Implication                                                                                                                                                             |
| :--------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Self-Signed Certificates** | RDP uses self-signed certificates by default. Clients cannot distinguish between genuine and forged certificates, enabling potential interception despite encryption warnings. |
| **Inadequate Encryption**    | Systems may not insist on TLS/SSL and might accept weak RDP Security encryption.                                                                                               |
| **Default WinRM (HTTP)**     | WinRM often defaults to HTTP (port 5985) instead of HTTPS (port 5986), sending management traffic in the clear.                                                                |
| **NLA Disabled**             | If Network Level Authentication (NLA) is not enforced, it may allow for easier footprinting and initial connection attempts.                                                   |
