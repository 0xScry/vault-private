# Metasploit: Session and Job Management

Metasploit provides the flexibility to manage multiple modules simultaneously through **Sessions** and **Jobs**, allowing for persistent control and background execution of tasks.

## Session Management

**Sessions** are dedicated control interfaces for deployed modules. When a session is placed in the background, the connection to the target host persists, allowing you to switch between different modules without losing access.

### Interacting with Sessions

|Command|Purpose|
|:--|:--|
|`[CTRL] + [Z]`|**Backgrounds** the current session from the prompt.|
|`background`|Meterpreter-specific command to **background** the session.|
|`sessions`|Lists all **active sessions**, including ID, type, and connection info.|
|`sessions -i <SESSION_ID>`|**Interacts** with the specified session ID.|

### Operational Workflow: Transitioning to Post-Exploitation

Use this workflow when you have established a stable communication channel and need to run additional tools (e.g., scanners or credential gatherers) on the compromised host.

1. **Background** the successful exploit session using `[CTRL] + [Z]` or `background`.
2. **Search** for the desired post-exploitation or auxiliary module.
3. **Load** the module and view requirements via `show options`.
4. **Set the session number** (e.g., `set SESSION <SESSION_ID>`) to link the module to your backgrounded session.
5. **Run** the module to execute the task on the target.

**Note:** Common **post-exploitation** archetypes include local exploit suggesters, internal network scanners, and credential gatherers.

---

## Job Management

**Jobs** are tasks running in the background. This is critical for maintaining listeners or active exploits that must run independently of your current terminal interaction.

### Why Use Jobs?

- **Port Management:** If you terminate a session using `[CTRL] + [C]`, the port may remain in use. Use the `jobs` command to terminate the specific task and **free the port** for a different module.
- **Persistence:** Tasks converted into jobs run seamlessly in the background, even if a specific session disappears.

### Command Reference: Jobs

|Command|Purpose|
|:--|:--|
|`exploit -j`|Launches an exploit as a **background job**.|
|`run -j`|Launches an auxiliary module as a **background job**.|
|`jobs -l`|**Lists** all currently running background jobs.|
|`jobs -h`|Displays the **help menu** for job manipulation.|
|`kill <JOB_ID>`|**Terminates** a specific job by its index number.|
|`jobs -K`|**Kills** all currently running jobs.|

### Execution Example

When starting a handler to listen for incoming connections, running it as a job ensures your terminal remains free for other tasks.

```
msf6 exploit(multi/handler) > exploit -j
[*] Exploit running as background job <JOB_ID>.
[*] Started reverse TCP handler on <ATTACK_IP>:<PORT>
```

### Active Job Listing Reference

Use `jobs -l` to identify which payloads are tied to specific ports.

```
msf6 exploit(multi/handler) > jobs -l

### Jobs

Id  Name                    Payload                    Payload opts
--  ----                    -------                    ------------
0   Exploit: multi/handler  generic/shell_reverse_tcp  tcp://<ATTACK_IP>:<PORT>
```