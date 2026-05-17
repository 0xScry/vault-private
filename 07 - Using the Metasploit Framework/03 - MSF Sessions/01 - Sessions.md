## METHODOLOGY

1. If a stable communication channel is established, background the session to free the console for additional modules.
2. Identify the session ID via the active sessions list.
3. To perform post-exploitation (gathering credentials, internal scanning), select a module and bind it to the existing session ID.
4. If a module requires a port currently occupied by another task, use job management to terminate the conflicting process.
5. For persistent listeners or exploits that should run without blocking the console, execute them as background jobs.

---

## Session Management

When to use: Managing multiple active module interfaces or switching from a foothold to post-exploitation while maintaining a persistent connection.

List all active communication channels to identify target IDs

```
sessions
```

Interact with a specific established session

```
sessions -i <SESSION_ID>
```

Background a Meterpreter session from within the channel

```
background
```

Background any active session via keyboard shortcut

```
[CTRL] + [Z]
```

**Gotchas** **Sessions can die** if the communication channel tears down during payload runtime.

## Post-Exploitation Execution

When to use: Running local exploit suggesters, credential gatherers, or network scanners after establishing a stable session.

1. Background the current session to return to the MSF prompt.
2. Select the desired post-exploitation module.
3. Set the session target in the module options.
4. Execute the module against the existing channel.

## Background Jobs

When to use: Running exploits or handlers that must persist in the background or when a port must be freed from a previous task.

Run an exploit or handler as a background task immediately

```
exploit -j
```

List all currently active tasks running in the background

```
jobs -l
```

Terminate a specific job by its index to free up system ports

```
kill <JOB_ID>
```

Terminate every active background job simultaneously

```
jobs -K
```

**Gotchas** **Ports will remain in use** and block new modules if an active exploit is terminated with `[CTRL] + [C]` instead of being killed via the jobs command.