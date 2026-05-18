## RDP Footprinting

TCP 3389 is open and NLA status, hostname, or OS version details are needed for initial enumeration.

Enumerate encryption settings and NTLM info using Nmap scripts

```
nmap -sV -sC <TARGET_IP> -p 3389 --script rdp*
```

Trace RDP cookies and packet contents to verify what security appliances might see

```
nmap -sV -sC <TARGET_IP> -p 3389 --packet-trace --disable-arp-ping -n
```

_Dangerous / misconfigured settings_

- **RDP Security** layer used instead of TLS/SSL, allowing inadequate encryption.
- **Self-signed certificates** configured by default, enabling potential man-in-the-middle attacks as clients cannot distinguish them from forged ones.

_Gotchas_

- **mstshash=nmap** cookie used by Nmap is a known signature that can be used by EDR to identify and lock out testers.

## RDP Security Auditing

Unauthenticated identification of supported RDP security layers and encryption methods is required.

Perform unauthenticated handshake analysis to determine protocol support

```
./rdp-sec-check.pl <TARGET_IP>
```

## RDP Interaction

Valid credentials have been obtained and GUI-based access to the server is the objective.

Initiate an interactive RDP session from a Linux host

```
xfreerdp /u:<USERNAME> /p:"<PASSWORD>" /v:<TARGET_IP>
```

## WinRM Footprinting

TCP 5985 or 5986 appear open during scanning, indicating potential for command-line management.

Identify the WinRM service and check the HTTP server header

```
nmap -sV -sC <TARGET_IP> -p 5985,5986 --disable-arp-ping -n
```

Confirm WS-Management reachability from a Windows-based pivot or workstation

```
Test-WsMan <TARGET_IP>
```

_Dangerous / misconfigured settings_

- **HTTP (5985)** protocol usage instead of HTTPS (5986), often occurring because port 80/443 were blocked or for legacy reasons.

## WinRM Interaction

Remote command execution is required from a Linux environment using valid credentials.

Establish a remote PowerShell session

```
evil-winrm -i <TARGET_IP> -u <USERNAME> -p <PASSWORD>
```

## WMI Interaction

Port 135 is open and a semi-interactive shell is needed through the Windows Management Instrumentation interface.

Execute commands via the WMI interface using Impacket

```
wmiexec.py <USERNAME>:"<PASSWORD>"@<TARGET_IP> "<COMMAND>"
```

_Edge cases_

- **Random ports** are utilized for the bulk of WMI communication after the initial connection is established on TCP 135.

> ⚠️ Gap: WMI requires that both **TCP 135** and the **random high ports** assigned after the handshake are reachable through any network firewalls, otherwise the command execution will time out silently.