## Intruder Request Setup

Captured request in Proxy History requires manual fuzzing of directories, parameters, or sub-domains.

Send highlighted request to Intruder

```
CTRL+I
```

Switch focus to Intruder tab

```
CTRL+SHIFT+I
```

## Payload Positioning

Request is loaded in Intruder and specific strings or headers need iteration.

Wrap targeted text with payload markers

```
§DIRECTORY§
```

**Gotchas**: **Missing trailing newlines** at the end of the request can cause the server to return error responses.

## Payload Type Selection

Position markers are defined and a source for payload data is required.

Use for standard wordlist files

```
Payload Type -> Simple List -> Load -> <FILE_PATH>
```

Use for massive wordlists to prevent memory exhaustion

```
Payload Type -> Runtime file
```

**Tool comparison**:

- Burp Community -> throttled at 1 request per second -> use only for short queries.
- Burp Pro -> unlimited speed -> use for production-scale fuzzing and brute-forcing.
- CLI Fuzzers (ffuf, gobuster, wfuzz) -> up to 10k requests per second -> prefer over Community for large-scale directory/sub-domain enumeration.
- ZAP Fuzzer -> unthrottled -> use as a free alternative for large attacks when Pro is unavailable.

## Payload Processing Rules

Loaded wordlist requires transformations like adding extensions or filtering junk characters.

Regex rule to skip lines starting with a dot

```
Skip if matches regex: ^..*$
```

Standard URL-encoding configuration for special characters

```
Payload Encoding -> URL-encode: ./^=<>&+?*:;'{}|^
```

**Edge cases**: Use Payload Processing to append extensions or filter wordlists based on specific criteria without modifying the original file.

## Attack Settings and Grep

High-volume results require automated flagging of success conditions or data extraction.

Flag responses containing specific success strings like status codes

```
Grep - Match -> Clear -> Add "200 OK"
```

Isolate and display specific parts of a response when content is lengthy

```
Grep - Extract
```

**Dangerous / misconfigured settings**:

- **Exclude HTTP Headers** enabled in Grep - Match when searching for status codes, as these reside in the header.
- Resource Pool left at default for very large attacks that require network resource capping.

**Gotchas**: **Throttled Community speed** makes the tool impractical for password spraying against Active Directory or large-scale brute-forcing.