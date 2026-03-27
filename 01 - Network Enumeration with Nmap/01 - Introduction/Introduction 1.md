
---

**Enumeration**

Enumeration is the most critical phase — the goal isn't just getting access, it's identifying _all possible attack vectors_.

Tools alone aren't enough. You need to understand how services work and what they expose. Most valuable info comes from **misconfigurations** and **neglected security**.

Two things to look for in every service:

- Functions/resources that allow interaction or leak info
- Info that leads to more info

**Manual enumeration matters** — scanners have timeouts and can mark ports as closed when they're just slow. A missed port = a missed attack path.

> The analogy: "keys in the living room" vs "white shelf, next to the TV, third drawer" — specificity is everything.

---

**Nmap**

Open-source network scanner written in C/C++/Python/Lua.

**What it does:**

- Host discovery
- Port scanning
- Service/version detection
- OS detection
- Scripting via NSE (Nmap Scripting Engine)

**Syntax:**

```bash
nmap <scan types> <options> <target>
```

**Key scan types:**

|Flag|Type|
|---|---|
|`-sS`|TCP SYN (default, stealthy)|
|`-sT`|TCP Connect|
|`-sU`|UDP|
|`-sA`|ACK|
|`-sN/sF/sX`|Null/FIN/Xmas|

**TCP SYN scan (`-sS`):**

- Sends SYN, never completes 3-way handshake
- `SYN-ACK` → port **open**
- `RST` → port **closed**
- No response → port **filtered**

```bash
sudo nmap -sS <target>
```