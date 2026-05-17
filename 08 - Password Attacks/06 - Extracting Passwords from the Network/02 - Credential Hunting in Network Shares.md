## Methodology

1. Access established to an internal environment where shared folders likely contain plaintext credentials or configuration files.
2. If on a domain-joined Windows host:
    - Run Snaffler to automatically discover shares and interesting files via AD.
    - Run PowerHuntShares if an HTML report and permission analysis are required.
3. If on Linux or a non-domain Windows host:
    - Use MANSPIDER via Docker to scan remote shares for specific content patterns.
    - Use NetExec with the spider module to target specific shares or search for content patterns across the target.
4. If automated tools are restricted:
    - Execute manual PowerShell searches using recursion and pattern matching.
5. Review all output for **false positives**.

---

## Windows Credential Hunting

### Manual Share Searching

Corporate environments use network shares for file storage; these often contain sensitive data like configuration files. Basic command-line searches provide a starting point before using automated tools.

Execute recursive search for specific extensions and patterns on a target share

```
Get-ChildItem -Recurse -Include *.<EXT> \\<TARGET_IP>\<SHARE_NAME> | Select-String -Pattern <PATTERN>
```

### Snaffler

A C# tool for domain-joined machines that automates share discovery and identifies interesting files.

Run basic automated search across the domain

```
Snaffler.exe -s
```

- **Tool comparison**
    
    - Snaffler
        - `Snaffler.exe -s`
        - Prefer when on a domain-joined machine and full domain computer/share discovery is needed.
    - PowerHuntShares
        - `Invoke-HuntSMBShares`
        - Prefer when an HTML summary report or detailed permission/risk analysis is required.
- **Gotchas**
    
    - **False positives** are common and require manual review of the output.

### PowerHuntShares

A PowerShell script that enumerates domain computers, checks for open SMB ports, and identifies shares with excessive privileges.

Automate share enumeration and generate an HTML report

```
Invoke-HuntSMBShares -Threads 100 -OutputDirectory <FILE_PATH>
```

- **Gotchas**
    - **Execution time** can reach several hours in large environments.

---

## Linux Credential Hunting

### MANSPIDER

A tool for scanning SMB shares remotely from Linux, best executed via Docker to manage dependencies.

Search for specific content patterns across remote shares with valid credentials

```
docker run --rm -v ./manspider:/root/.manspider blacklanternsecurity/manspider <TARGET_IP> -c '<PATTERN>' -u '<USERNAME>' -p '<PASSWORD>'
```

### NetExec

A multi-purpose tool that includes spidering capabilities to search through network shares.

Spider a specific share for files containing a specific pattern

```
nxc smb <TARGET_IP> -u <USERNAME> -p '<PASSWORD>' --spider <SHARE_NAME> --content --pattern "<PATTERN>"
```

- **Tool comparison**
    
    - MANSPIDER
        - `docker run ... blacklanternsecurity/manspider <TARGET_IP> -c '<PATTERN>'`
        - Prefer for remote scans from Linux when loot needs to be downloaded locally.
    - NetExec
        - `nxc smb <TARGET_IP> --spider <SHARE_NAME> --content`
        - Prefer for quick spidering of specific shares or when already using NetExec for other protocol tasks.
- **Gotchas**
    
    - **Incomplete results** may occur if searching by file content without the `--content` flag in NetExec.