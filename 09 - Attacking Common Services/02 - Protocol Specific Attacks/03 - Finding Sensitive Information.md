1. Identify and enumerate low-security services like FTP or Storage to find initial leads.,
2. Document every string, filename, or metadata point, even if it appears insignificant or doesn't work on the local service.,
3. Re-purpose discovered strings as credentials for communication services like email.
4. Interrogate email archives specifically for high-value terms like "password" to find credentials for critical backend services.
5. Authenticate to databases (e.g., MSSQL) using retrieved credentials and leverage internal features to achieve RCE.

---

## Service Metadata Harvesting

When to use: Anonymous access is available on a service like FTP and you need to build a username or credential list.

Commands

> ⚠️ Gap: Source does not provide CLI syntax for anonymous login or directory listing.

Gotchas **Ignoring single filenames** or metadata points can stall an engagement; strings that fail locally often serve as valid credentials for secondary services like email,.

## Communication Service Interrogation

When to use: You have identified potential usernames/passwords and need to find credentials for high-tier services like databases.

Commands

> ⚠️ Gap: Source does not provide tool syntax or protocols for authenticating to and searching email services.

## Database Command Execution

When to use: You have acquired valid MSSQL credentials and require RCE on the database server.

Commands

> ⚠️ Gap: Source does not provide the specific SQL commands or "built-in functionality" syntax required to trigger command execution.

Edge cases **Target-specific business models** and processes dictate what information is considered "sensitive"; understanding the client's purpose changes what data you prioritize during searches.