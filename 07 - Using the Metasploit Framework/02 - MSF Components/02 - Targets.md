1. Run `info` to audit module functionality, check for **artifact generation**, and verify dependencies like ROP chain validity or specific JRE versions.
2. Run `show targets` to list supported operating systems and service packs.
3. Identify the remote OS/version and match it to the available target IDs.
4. Use `set target <ID>` for manual precision or `set target 0` for **Automatic** service detection.

---

## Module Auditing

Verify exploit functionality, platform requirements, and environmental dependencies before execution.

View module metadata, ROP chain requirements, and mandatory software versions

```
info
```

- **Audit code** for artifact generation to ensure a clean working environment.
- Check for specific requirements like `msvcrt` on XP SP3 or `JRE 1.6.x` on Windows 7 for valid ROP chains.

## Target Identification and Selection

Align the exploit with unique operating system identifiers and return addresses like `jmp esp` or `pop/pop/ret`.

List all available vulnerable targets for the active module

```
show targets
```

Manually set the specific target version by index number

```
set target <ID>
```

- **Automatic** target (Id 0) triggers service detection before launching the attack.
    
- Manual selection is required when language packs or hooks shift return addresses.
    
- Running `show targets` in the root menu **fails**; an exploit module must be selected first.
    
- The **exploit fails** if the selected target ID does not match the target's specific OS version, service pack, or language-specific return address.