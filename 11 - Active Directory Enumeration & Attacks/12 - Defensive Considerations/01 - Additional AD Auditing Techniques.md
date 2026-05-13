## AD Database Snapshotting

Need to navigate the AD database or compare object attributes before and after changes. Any **valid domain user** credentials grant access to browse or save the database.

1. Connect to the domain using valid credentials
2. Navigate to File --> Create Snapshot
3. Enter a name for the snapshot and move it offline for analysis

**AD Explorer**

- [GUI Interaction]
- Prefer when requiring a portable, offline copy of the directory for reporting or comparing attribute changes over time.

**Gotchas** **Invalid domain credentials** prevent tool connection to the DC.

## AD Healthcheck and Risk Assessment

Target needs a host inventory, visual domain maps, or maturity-based risk scoring. Minimum access requires a network path to a DC and standard user context.

Launch interactive TUI for menu-driven auditing

```
PingCastle.exe
```

Execute with specific credentials and protocol preference

```
PingCastle.exe --server <DC_IP> --user <USERNAME> --password <PASSWORD> --protocol ADWSOnly
```

- **PingCastle** → `PingCastle.exe --interactive` → Prefer for CMMI risk scoring and host inventory maps.
- **BloodHound / PowerView** → Prefer for identifying specific attack paths and granular object enumeration.

**Gotchas** **System date after July 31, 2023** prevents the tool from starting. **Mass workstation scanning** via the scanner menu will likely trigger security alerts.

## AD Security Scanning

Individual workstation checks or specific vulnerability lookups are required after a baseline is established. Accessed via the `Scanner` option in the main menu.

Run specific workstation security checks (e.g., Zerologon, LAPS, SMB)

```
PingCastle.exe --scanner <SERVICE_NAME> --server <DOMAIN>
```

**Dangerous / misconfigured settings**

- Null sessions and null session trusts
- Lack of LAPS or BitLocker coverage
- Spooler service enabled on sensitive hosts

**Gotchas** **Missing -f or -s flag** in related GPO tools or output redirection causes immediate execution failure.

## Group Policy Auditing

Targeting GPO misconfigurations like registry-based settings or user rights assignments. Must run from a domain-joined host or via `runas /netonly` as a domain user.

Run audit and redirect output to a log file

```
group3r.exe -f <FILE_PATH>
```

Run audit and output results directly to stdout

```
group3r.exe -s
```

**Edge cases**

- Use `-h` to find additional flags if standard file output fails.

**Gotchas** **Missing -f or -s flag** causes the tool to fail without processing any data.

## Automated AD Data Collection

Stealth is not a requirement and a full dump of forest details, LAPS, and DNS is needed for broad analysis.

Execute main collection script

```
.\ADRecon.ps1
```

Regenerate Excel report from existing CSV data on a host with Excel installed

```
.\ADRecon.ps1 -GenExcel -Path <FILE_PATH>
```

**Gotchas** **Excel not installed** on the execution host prevents automatic report generation, leaving only raw CSVs. **GroupPolicy PowerShell module missing** prevents the script from gathering GPO data.

> ⚠️ Gap: ADRecon will fail to collect GPO information silently if the GroupPolicy module is not present on the host where the script is executed.