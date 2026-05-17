## Analyzing Cleartext Traffic with Wireshark

Identifying exposed **credentials** such as usernames, passwords, and community strings in unencrypted legacy or misconfigured services like HTTP, FTP, or SNMP.

Filter for all HTTP traffic

```
http
```

Isolate login attempts potentially containing sensitive data in the request body

```
http.request.method == "POST"
```

Search for specific keywords within the HTTP protocol payload

```
http contains "passw"
```

Follow a specific conversation between two hosts to reconstruct the session

```
tcp.stream eq <PORT>
```

Identify SYN packets to detect scanning or connection attempts

```
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

Monitor DNS resolution for reconnaissance

```
dns
```

Filter traffic between two specific target hosts

```
ip.src == <TARGET_IP> && ip.dst == <PIVOT_IP>
```

- Services utilizing **unencrypted protocols** (HTTP, FTP, SNMPv1/v2, POP3, IMAP, SMTP, LDAP, SMB, VNC) transmit data in cleartext.
    
- **TLS/SSL encryption** will render credentials unreadable unless a decryption method is applied to the capture.
    

## Automated Credential Extraction with Pcredz

Rapidly parsing large packet captures or live traffic for **NTLM hashes**, **Kerberos tickets**, and protocol-specific credentials without manual filtering.

Run Pcredz against a capture file with verbose output and timestamps

```
./Pcredz -f <FILE_PATH> -t -v
```

- Wireshark -> `http.request.method == "POST"` -> prefer for manual session reconstruction and deep protocol analysis.
    
- Pcredz -> `./Pcredz -f <FILE_PATH> -t -v` -> prefer for automated, multi-protocol extraction of hashes and community strings.
    
- **Missing dependencies** or library version mismatches will cause the tool to fail; use the provided Docker container to ensure a stable environment.