## Methodology

1. Confirm target is a **Windows** host running **ASP.NET**.
2. Identify an upload vector or file write primitive that allows **ASPX** execution.
3. Prepare the Antak payload by stripping **signatures** and setting **credentials** to prevent unauthorized access.
4. Upload the modified payload to a web-accessible directory.
5. Authenticate via the browser to gain a PowerShell-themed console.
6. Use the shell's built-in functions to execute scripts in memory or pivot to a full C2 callback.

---

## WebShell Preparation

Target is a Windows IIS server and you require PowerShell-level interaction rather than basic CMD. Minimum conditions: write access to a directory where **ASPX** files are permitted to execute.

Copy the base shell to your working directory

```
cp /usr/share/nishang/Antak-WebShell/antak.aspx /home/<USERNAME>/<FILE_PATH>.aspx
```

Modify line 14 to set access credentials

```
if ($username -eq "<USERNAME>" -and $password -eq "<PASSWORD>")
```

**Dangerous / misconfigured settings**

- Leaving default **ASCII art** and comments in the file: these are frequently **signatured** by AV/EDR.
- Using default or weak **credentials** on the shell: allows third parties to hijack the execution point.

**Gotchas** **AV/EDR detection** will likely trigger if the file is uploaded without stripping Nishang-specific comments and metadata.

## Shell Execution and Interaction

You have successfully uploaded the shell and need to bypass standard process limitations. Antak is required when you need to **encode commands** or execute PowerShell scripts directly in memory.

Access the shell via browser

```
http://<TARGET_IP>/<FILE_PATH>.aspx
```

Built-in command to view available functions

```
help
```

**Edge cases**

- Script execution in memory: Use the **Encode and Execute** button to bypass basic command-line logging or simple filters.
- Connection string discovery: Use the **Parse web.config** button if you land in a web root to extract database credentials.

**Gotchas** **New process per command** means environment variables or directory changes (`cd`) do not persist between separate submissions.

## File Operations and SQL

Symptom: You have a web shell but need to move tools onto the target or query the local database. Minimum conditions: the service account has write permissions to the current directory or the SQL connection string is known.

Upload a local file to the target through the browser UI

```
[Browse] -> [Upload the File]
```

Execute raw SQL queries via the connection string field

```
SELECT * FROM <SERVICE_NAME>
```

**Gotchas** **File upload failures** often occur because the IIS worker process account lacks write permissions to the specific subdirectory where the shell is located.