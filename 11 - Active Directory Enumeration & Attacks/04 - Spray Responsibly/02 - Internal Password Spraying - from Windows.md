## Internal Windows Password Spraying

Foothold on domain-joined host or physical access to managed Windows device; requires domain authentication or a pre-generated user list.

Importing module for current session

```
Import-Module .\DomainPasswordSpray.ps1
```

Standard spray leveraging AD discovery and automated lockout protection

```
Invoke-DomainPasswordSpray -Password <PASSWORD> -OutFile <FILE_PATH> -ErrorAction SilentlyContinue
```

Spray using external user list when not authenticated to the domain

```
Invoke-DomainPasswordSpray -UserList <FILE_PATH> -Password <PASSWORD> -OutFile <FILE_PATH>
```

Successful spray output identifying valid accounts

```
[ *] SUCCESS! User:<USERNAME> Password:<PASSWORD>
```

- DomainPasswordSpray -> `Invoke-DomainPasswordSpray` -> prefer for domain-joined hosts to auto-filter users near lockout threshold.
    
- Kerbrute -> `C:\Tools\kerbrute.exe` -> prefer for targeting **LDAP** or **Kerberos pre-authentication** to evade **SMB** specific monitoring.
    
- Applications allowing login for any domain user regardless of role or necessity.
    
- MFA implementations that confirm credential validity even if they block the login, allowing for credential reuse elsewhere.
    
- Target **LDAP** or **Kerberos pre-authentication** (Event ID 4771) to avoid noise in **SMB** security logs (Event ID 4625).
    
- **Denial of Service** occurs when restrictive lockout policies trigger and require manual admin intervention after a careless spray.
    

---

## Spray Detection and Evasion

Seeing high lockout rates or needing to identify specific logging triggers during active engagement.

- Restrictive lockout policies requiring manual administrative intervention.
    
- **Event ID 4625**: Indicates multiple account logon failures over **SMB**.
    
- **Event ID 4771**: Indicates **Kerberos pre-authentication** failures, typically from **LDAP** spraying.
    
- **Detection** occurs when organizations correlate many logon failures within a specific time interval to trigger alerts.