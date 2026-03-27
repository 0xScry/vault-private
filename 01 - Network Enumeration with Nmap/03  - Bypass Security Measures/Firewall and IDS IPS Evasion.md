**Nmap — Firewall & IDS/IPS Evasion**

**ACK Scan (`-sA`)** — harder to filter than SYN/Connect because firewalls often can't tell if ACK packets belong to an existing connection.

```bash
sudo nmap 10.129.2.28 -p 21,22,25 -sA -Pn -n --disable-arp-ping --packet-trace
```

- Open/closed port → returns `RST`
- Filtered → no response or ICMP unreachable
- Result shows `unfiltered` instead of `open` — useful to map what the firewall is blocking vs passing

---

**Decoys (`-D`)** — spoof multiple source IPs to hide your real one

```bash
sudo nmap 10.129.2.28 -p 80 -sS -Pn -n --disable-arp-ping -D RND:5
```

- `RND:5` = 5 random IPs + your real IP mixed in
- Decoys must be alive — dead IPs can trigger SYN-flood protection
- Works with SYN, ACK, ICMP, OS detection scans

**Spoof source IP (`-S`)** — test if a specific subnet has access

```bash
sudo nmap 10.129.2.28 -p 445 -O -S 10.129.2.200 -e tun0 -Pn -n
```

---

**DNS Proxying / Source Port abuse**

Firewalls often trust traffic from port 53 (DNS). If a port is filtered:

```bash
# Confirm filtered
sudo nmap 10.129.2.28 -p 50000 -sS -Pn -n --disable-arp-ping

# Try from source port 53
sudo nmap 10.129.2.28 -p 50000 -sS -Pn -n --disable-arp-ping --source-port 53
```

If it opens → connect with ncat the same way:

```bash
ncat -nv --source-port 53 10.129.2.28 50000
```

Custom DNS server:

```bash
--dns-server <ns>   # use internal DNS (more trusted in DMZ)
```

---

**Detecting IDS/IPS:**

- IDS = passive, alerts admin
- IPS = active, blocks automatically
- To detect: scan aggressively from a VPS — if your IP gets blocked, IPS is present
- Switch to a different VPS and scan quieter