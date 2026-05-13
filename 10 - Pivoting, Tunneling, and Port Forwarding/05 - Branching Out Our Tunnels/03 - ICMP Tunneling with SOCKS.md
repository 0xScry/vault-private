## ICMP Tunneling with ptunnel-ng

Egress filtering blocks standard TCP/UDP outbound but allows ICMP echo requests to external servers.

Build a static binary to ensure compatibility across different Linux distributions

```
sudo apt install automake autoconf -y
cd ptunnel-ng/
sed -i '$ s/.*/LDFLAGS=-static "${NEW_WD}/configure" --enable-static  $@ \&\& make clean \&\& make -j$ {BUILDJOBS:-4} all/' autogen.sh
./autogen.sh
```

Start the server-side listener on the target pivot host

```
sudo ./ptunnel-ng -r<PIVOT_IP> -R<SERVICE_PORT>
```

Establish the client-side tunnel on the attack host to map a local port to the remote service

```
sudo ./ptunnel-ng -p<PIVOT_IP> -l<LOCAL_PORT> -r<PIVOT_IP> -R<SERVICE_PORT>
```

- **GLIBC version mismatch** will prevent the binary from executing if the build environment and target environment are not aligned.
- **Missing root privileges** on either side prevents ptunnel-ng from opening the raw sockets required for ICMP packet manipulation.

> ⚠️ Gap: The source specifies using `sed` to force a static build in `autogen.sh` but does not provide a fallback if the target environment lacks the necessary headers or libraries to compile even a static binary.

## SSH Pivoting and Dynamic Forwarding

The ICMP tunnel is active on a local port and you need to pivot into the internal network using standard tools.

Connect to the target via the ICMP-encapsulated SSH tunnel

```
ssh -p<LOCAL_PORT> -l<USERNAME> 127.0.0.1
```

Enable dynamic port forwarding through the tunnel to use with proxychains

```
ssh -D <SOCKS_PORT> -p<LOCAL_PORT> -l<USERNAME> 127.0.0.1
```

Execute internal scans via proxychains through the established ICMP-SSH chain

```
proxychains nmap -sV -sT <TARGET_IP> -p<PORT>
```

- Specify the reachable IP of the jump-box with the `-r` flag on the server side to ensure the listener binds to the correct interface.
    
- **TCP/SSHv2 traffic detection** is bypassed at the network level as packet analyzers will only see ICMP echo traffic.
    
- **Incomplete lab setup** requires waiting 3-5 minutes after spawning a target before the tunnel will function.