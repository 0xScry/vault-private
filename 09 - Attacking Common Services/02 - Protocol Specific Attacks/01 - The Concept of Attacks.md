# Methodology

1. **Analyze Source**: Identify where information enters the task cycle (Code, Libraries, Config, APIs, or User Input),.
2. **Evaluate Process**: Map how data is handled by looking at PID privileges, input functions, variables, and logging triggers.
3. **Check Privileges**: Determine the execution context (System/root, User, Groups, Policies, or Rules) to define the impact.
4. **Identify Destination**: Track where data is stored or forwarded (Local files or Network interfaces) to confirm the exploit goal.
5. **Cycle Iteration**: If the Destination provides new information to a subsequent process, repeat the analysis for multi-stage attacks,.

---

## Source Manipulation

Identifying and exploiting the entry point where information is passed to a process.

Modify specific request headers to pass malicious commands to internal libraries,

```
User-Agent: <JNDI_LOOKUP>
```

- **Dangerous / misconfigured settings**
    
    - Processing unsanitized manual entry via User Input.
    - Using results from previously executed program Code as a trusted information source.
    - Relying on static or prescribed values in Config files that determine process behavior.
- **Edge cases**
    
    - Vulnerabilities in Libraries (e.g., Log4j) can affect any software product integrating their classes and functions.
- **Gotchas**
    
    - **Destination as new Source** occurs when the initial task completion serves as the starting point for a subsequent malicious cycle,.

---

## Process & Privilege Analysis

Determining how information is misinterpreted and the permissions assigned to the executing process,.

Analyze the data processing logic to find where input is handled as executable code rather than strings,

```
<DATA_PROCESSING_FUNCTION>
```

- **Dangerous / misconfigured settings**
    
    - Administrative or **SYSTEM/root** privileges assigned to logging or sensitive data processes,.
    - Misinterpretation of strings that leads to the execution of a request.
    - Hard-coded functions in Data processing that fail to validate variables.
- **Gotchas**
    
    - **Privilege mismatch** occurs if the process does not satisfy the conditions required by the system, causing the requested action to be rejected.
    - **Admin logging permissions** often allow user-supplied code to execute with the highest level of system control.

---

## Destination Exploitation

Confirming the final target or forwarding point of the manipulated task.

Redirect process output or requests to a remote server for RCE,

```
<TARGET_IP>:<PORT>
```

- **Edge cases**
    
    - Destination can be Local (files/records) or Network-based (remote interfaces/IPs).
- **Gotchas**
    
    - **Network-to-Source loop** triggers when a remote query retrieves a malicious class that is then used as a source for further process execution,.

> ⚠️ Gap: Source lacks specific tools or syntax for intercepting/modifying HTTP headers (e.g., Burp Suite) and lacks instructions for standing up the remote listener/LDAP server required to complete the JNDI lookup.