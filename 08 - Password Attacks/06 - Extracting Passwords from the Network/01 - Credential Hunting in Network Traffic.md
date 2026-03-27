### Credential Hunting in Network Traffic

**Credential hunting** in network traffic targets **legacy systems**, **misconfigured services**, or test applications that utilize unencrypted protocols. While TLS is standard, gaps in security allow for the identification of cleartext usernames, passwords, and sensitive strings within captured traffic.

---

### Protocol Identification

Identify target services by looking for **unencrypted protocols** that have not been migrated to their secure counterparts.

|Unencrypted Protocol|Encrypted Counterpart|Description|
|:--|:--|:--|
|**HTTP**|HTTPS|Web pages and resources.|
|**FTP**|FTPS/SFTP|File transfers.|
|**SNMP** (v1/v2)|SNMPv3 (with encryption)|Network device monitoring/management.|
|**POP3**|POP3S|Email retrieval.|
|**IMAP**|IMAPS|Email management on server.|
|**SMTP**|SMTPS|Email sending.|
|**LDAP**|LDAPS|Directory services and roles.|
|**RDP**|RDP (with TLS)|Windows remote desktop access.|
|**DNS** (Traditional)|DNS over HTTPS (DoH)|Domain name resolution.|
|**SMB**|SMB over TLS (SMB 3.0)|File and resource sharing.|
|**VNC**|VNC with TLS/SSL|Graphical remote control.|

---

### Wireshark Analysis Methodology

Use **Wireshark** to analyze live or captured traffic. The primary goal is to isolate high-value conversations or specific packets containing authentication data.

#### 1. Traffic Filtering

Apply display filters to narrow the scope to relevant protocols or hosts.

|Wireshark Filter|Description / Use Case|
|:--|:--|
|`ip.addr == <TARGET_IP>`|Isolate all traffic involving a specific host.|
|`tcp.port == <PORT>`|Filter by specific service port (e.g., 80 for HTTP).|
|`http`|View all HTTP protocol traffic.|
|`dns`|Monitor domain name resolution attempts.|
|`tcp.flags.syn == 1 && tcp.flags.ack == 0`|Detect **scanning** or new connection attempts.|
|`icmp`|Isolate Ping traffic for reconnaissance or troubleshooting.|
|`http.request.method == "POST"`|**High Value**: Isolate data submissions where passwords may reside.|
|`tcp.stream eq <STREAM_ID>`|Follow a specific conversation between two hosts.|
|`eth.addr == <MAC_ADDRESS>`|Filter by hardware address.|
|`ip.src == <ATTACK_IP> && ip.dst == <TARGET_IP>`|Track communication between two specific hosts.|

#### 2. String Searching

When dealing with large captures, use specific string searches to find credential-related keywords like "password" or "user".

- **Filter Method**: Use `http contains "<KEYWORD>"` (e.g., `http contains "passw"`) to find packets with matching data.
- **Manual Search**: Navigate to **Edit > Find Packet** and search for specific strings manually.

---

### Automated Extraction with Pcredz

**Pcredz** is used to rapidly scan network traffic for credentials, supporting both live interfaces and packet capture files. It is particularly effective for extracting **NTLM** (HTTP, LDAP, SMB, MSSQL, RPC), **Kerberos**, **FTP**, and **HTTP Basic** authentication data.

#### Operational Workflow

1. **Identify Capture File**: Ensure the packet capture (e.g., `.pcapng`) is available.
2. **Run Extraction**: Execute the tool with verbose logging to identify community strings and credentials.

```
./Pcredz -f <FILENAME> -t -v
```

**Parameters:**

- `-f`: Specifies the input packet capture file.
- `-t`: Enables TCPDump format parsing (used if the format is initially unknown).
- `-v`: Enables verbose output to see findings in real-time.

#### Attack Implications

- **SNMP**: Extraction of **Community Strings** (e.g., SNMPv2) can lead to further network reconnaissance or device reconfiguration,.
- **FTP**: Cleartext extraction of **User** and **Pass** fields provides immediate access to file shares.