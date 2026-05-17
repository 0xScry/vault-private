## Shell Identification

When to use: **Command execution fails with unrecognized syntax** or the session prompt is ambiguous; identifying the interpreter is required to determine valid command and scripting syntax.

Check the process list for the active shell binary

```
ps
```

Inspect environment variables for the shell path

```
env
```

Force a signature-specific error message

```
<RANDOM_STRING>
```

- `ps` -> `ps` -> prefer for verifying the actual binary processing input in the current session.
    
- `env` -> `env` -> prefer for identifying the user's default configured shell path.
    
- **Cross-platform interpreters**: PowerShell can be present on Linux systems, and its presence is not restricted to Windows environments.
    
- **Decoupled interfaces**: Terminal emulators are not tied to specific languages and can be reconfigured to launch any interpreter.
    

**Prompt cue ambiguity** occurs when assuming a shell type based on the `$` symbol, as it is used by Bash, Ksh, POSIX, and others.