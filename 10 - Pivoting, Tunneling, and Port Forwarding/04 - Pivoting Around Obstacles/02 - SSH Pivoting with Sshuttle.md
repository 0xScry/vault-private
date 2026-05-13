## SSH Subnet Routing

Internal network resources are unreachable and manual **proxychains** encapsulation is breaking tool workflows; requires SSH access and **Python** on the remote pivot.,

Install the utility on the attack host:

```
sudo apt-get install sshuttle
```

Automate **iptables** rules to route a specific internal subnet through the SSH tunnel:

```
sudo sshuttle -r <USERNAME>@<PIVOT_IP> <TARGET_IP>/<MASK> -v
```

- **Sshuttle** -> `sshuttle -r <USERNAME>@<PIVOT_IP> <TARGET_IP>/<MASK>` -> Prefer for transparently routing **Nmap** or other tools natively without prefixing every command,,.
- **Proxychains** -> Prefer when the pivot requires **TOR** or **HTTPS** proxies, as these are unsupported by this tool.

**Gotchas** **Connection failure** occurs if the remote pivot host does not have a **Python** interpreter installed. **Traffic drop** will occur for **UDP** traffic when using the default **nat** method. **Pivot failure** results if attempting to use this tool for protocols other than **SSH**.