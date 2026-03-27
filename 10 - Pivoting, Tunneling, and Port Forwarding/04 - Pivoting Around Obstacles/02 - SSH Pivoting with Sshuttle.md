# SSH Pivoting with Sshuttle

**Sshuttle** is a Python-based tool used to automate the execution of **iptables** and add pivot rules for a remote host. Use this technique to route all network traffic through a pivot point without the need to configure or use **proxychains**. This technique specifically works for pivoting over **SSH** and does not support other options like TOR or HTTPS proxy servers. Establishing this pivot unlocks the ability to use any networking tool directly against internal targets as if they were on the local network.

### Operational Workflow

1. **Install the Tool**: Use the package manager to install **sshuttle** on the attack machine.
2. **Establish the Connection**: Connect to the pivot host using SSH credentials while specifying the internal subnet to be routed.
3. **Monitor Redirection**: Sshuttle automatically creates entries in the **iptables** to redirect traffic destined for the internal network through the pivot host.
4. **Direct Interaction**: Execute tools, such as **Nmap** or **RDP** clients, directly against internal target IP addresses.

### Command Reference

|Command|Description|
|:--|:--|
|`sudo apt-get install sshuttle`|Installs the **sshuttle** package.|
|`sudo sshuttle -r <USERNAME>@<PIVOT_IP> <INTERNAL_SUBNET> -v`|Connects to the pivot host and routes the specified internal network through it.|
|`sudo nmap -v -A -sT -p <PORT> <TARGET_IP> -Pn`|Scans an internal target directly via the established **sshuttle** route.|

### Parameter Reference

|Parameter|Description|
|:--|:--|
|`-r`|Specifies the remote connection string using `<USERNAME>@<PIVOT_IP>`.|
|`-v`|Enables **verbose** output to track the firewall manager and connection status.|
|`<INTERNAL_SUBNET>`|The network range (e.g., 172.16.5.0/23) that should be routed through the pivot.|

### Execution Notes

- **Automation**: Sshuttle automates the setup of **nat** methods and **iptables** rules (such as `PREROUTING` and `OUTPUT`) to handle TCP redirection.
- **Direct Access**: Once the connection is established, tools can be used directly against the internal network without prefixing them with proxy commands.