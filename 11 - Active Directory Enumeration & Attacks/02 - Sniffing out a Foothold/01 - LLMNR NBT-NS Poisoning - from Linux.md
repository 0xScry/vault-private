1. Confirm local network access and ensure **ports required by Responder** (UDP 137, 138, 53; TCP 80, 445, etc.) are not occupied by local services.
2. Run Responder in passive analysis mode to verify LLMNR or NBT-NS traffic is present on the segment.
3. Switch to active poisoning to spoof responses to failed DNS queries and capture NetNTLM hashes.
4. Monitor the console or check `/usr/share/responder/logs` for captured hash files.
5. Identify the hash type and execute an offline brute force attack to recover cleartext credentials.

---

## LLMNR/NBT-NS Poisoning

DNS resolution fails on a target machine, causing it to broadcast for host identification via UDP 5355 (LLMNR) or UDP 137 (NBT-NS).

Passive listening to identify resolution requests without sending poisoned packets

```
sudo responder -I <INTERFACE> -A
```

Active poisoning to spoof authoritative responses and capture authentication hashes

```
sudo responder -I <INTERFACE>
```

Force NTLM/Basic authentication on WPAD proxy requests to capture credentials

```
sudo responder -I <INTERFACE> -w -F
```

- Responder
    
    - `sudo responder -I <INTERFACE>`
    - Prefer for Linux attack hosts; purpose-built for LLMNR, NBT-NS, and MDNS.
- Inveigh
    
    - `Inveigh.exe`
    - Prefer for Windows-based MITM attacks; supports C# and PowerShell.
- Metasploit
    
    - `use auxiliary/spoof/llmnr/llmnr_response`
    - Prefer if already integrated into a Metasploit workflow.
- Disabling rogue servers (e.g., SMB, HTTP) in `Responder.conf` to avoid conflicting with other tools.
    
- Use `-r` or `-d` to answer NetBIOS wredir or domain suffix queries, though this is **likely to break network functionality**.
    
- Apply `-e <ATTACK_IP>` to poison requests with an IP other than the Responder host.
    
- **Missing sudo/root privileges** will prevent the tool from binding to necessary low-numbered ports.
    
- **Active ports on the attack host** (like a local SMB or web server) will cause Responder modules to fail.
    

## NetNTLM Hash Cracking

NetNTLMv1 or NetNTLMv2 hashes have been captured and saved to the local logs directory.

Dictionary attack against NetNTLMv2 hashes using the standard rockyou wordlist

```
hashcat -m 5600 <HASH_FILE> <WORDLIST>
```

- **NetNTLM hashes cannot be used for pass-the-hash**; they must be cracked or relayed to gain access.
- **Large or complex passwords** may be impossible to crack within assessment time limits due to the "slow" nature of the hash type.