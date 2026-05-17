## Searching for Modules

Identifying proof-of-concept exploits or scanning tools based on service versioning, CVE IDs, or target architecture.

Basic keyword search to display matching modules

```
search <SERVICE_NAME>
```

Filtered search to isolate reliable exploits for a specific platform and vulnerability year

```
search type:exploit platform:<OS> cve:<YEAR> rank:excellent <PATTERN>
```

Export search results to a CSV file for external documentation

```
search -o <FILE_PATH>
```

Search while excluding specific categories or terms

```
search <KEYWORD> -<KEYWORD>:<VALUE>
```

Sort results by a specific column such as rank or disclosure date

```
search <PATTERN> -s rank -r
```

- **Exploit failure** occurs frequently even when a vulnerability exists if the module is not customized for the specific target host.
- **Non-interactive modules** including Encoders, NOPs, and Payloads cannot be selected using the index number selection method.

## Module Configuration and Execution

Loading a selected module and defining the parameters required for payload delivery or auxiliary functions.

Select a module by its corresponding index number from the search results

```
use <INDEX_NO>
```

List required and optional parameters that must be configured before execution

```
show options
```

Retrieve comprehensive module metadata, author information, and technical references

```
info
```

Configure a variable for the current module context only

```
set <OPTION_NAME> <VALUE>
```

Define a persistent global variable to maintain values like target IPs across multiple modules

```
setg <OPTION_NAME> <VALUE>
```

Launch the configured exploit or initiate an auxiliary scanner

```
run
```

Drop into a standard operating system shell from an active Meterpreter session

```
shell
```

- **Check method** is not available for all modules; verify compatibility in the search results or module info before attempting to validate a vulnerability.
- **LHOST** must be accurately defined for all reverse TCP payloads or the connection will fail to call back to the attacker.