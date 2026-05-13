## Local Port Forwarding

Target service is bound to localhost on the compromised host or blocked by an external firewall; requires SSH access to the pivot.

Forward a single remote service to a local port to enable remote exploit execution or local tool access:

```
ssh -L <PORT>:localhost:<PORT> <USERNAME>@<PIVOT_IP>
```

Forward multiple services simultaneously by chaining the `-L` flag:

```
ssh -L <PORT>:localhost:<PORT> -L <PORT>:localhost:<PORT> <USERNAME>@<PIVOT_IP>
```

Confirm the local listener is active and identifying the forwarded service:

```
netstat -antp | grep <PORT>
```

```
nmap -v -sV -p<PORT> localhost
```

- **MySQL/Local services** may be inaccessible for remote exploitation if they are strictly bound to the target's loopback interface.

## Dynamic Port Forwarding (SOCKS Tunneling)

Pivot host has multiple NICs and internal subnets are not directly routable from the attack host.

Open a local SOCKS proxy to route traffic through the pivot to any reachable internal network:

```
ssh -D <PORT> <USERNAME>@<PIVOT_IP>
```

> ⚠️ Gap: The source recommends adding `socks4` to the configuration, but **SOCKS4** does not support UDP; if the internal service requires UDP, the connection will fail without a **SOCKS5** configuration.

## Proxychains Configuration

SOCKS tunnel is active but tools require a wrapper to force TCP traffic through the local listener.

Edit `/etc/proxychains.conf` to point to the active SSH tunnel:

```
tail -4 /etc/proxychains.conf
```

```
socks4 127.0.0.1 <PORT>
```

## Internal Subnet Enumeration

SOCKS tunnel and proxychains are configured but internal host availability and services are unknown.

Perform a ping scan to identify alive hosts in the internal range:

```
proxychains nmap -v -sn <TARGET_IP_RANGE>
```

Execute a full port scan on a specific internal host:

```
proxychains nmap -v -Pn -sT <TARGET_IP>
```

- **Partial packets** like half-connect/SYN scans return incorrect results; always use a **Full TCP connect scan** (`-sT`) over proxychains.
- **ICMP requests** are blocked by Windows Defender; use `-Pn` to prevent Nmap from marking internal Windows hosts as down.

## Internal Service Access

Internal services are identified through the tunnel and require direct interaction or exploitation.

Launch Metasploit and route all module traffic through the SOCKS proxy:

```
proxychains msfconsole
```

Establish an RDP session to an internal Windows target:

```
proxychains xfreerdp /v:<TARGET_IP> /u:<USERNAME> /p:<PASSWORD>
```

- **RDP certificates** must be accepted manually before the session initializes; failure to accept will prevent the connection through the tunnel.