1. Intercept request and identify encoded values in parameters, cookies, or headers.
2. Send to **Burp Decoder** or **ZAP Encoder/Decoder/Hash** to identify plaintext structure.
3. Modify logic in plaintext (e.g., flip boolean flags or change usernames).
4. Re-encode to the target format (Base64, URL, etc.).
5. Replace the original value in **Repeater** or **Request Editor** and submit.

---

## Request URL Encoding

Seeing **server error** responses or malformed data handling when modifying request parameters manually.

### Burp Suite

Keybind to URL-encode specific selection

```
CTRL+U
```

Context menu path for key characters

```
Convert Selection > URL > URL-encode key characters
```

Enable live encoding for manual entry in Repeater

```
Right-click > URL-encode as we type
```

- Burp Suite -> `CTRL+U` -> Prefer for targeted encoding of specific payload characters while leaving structure intact.

### OWASP ZAP

ZAP automates this in the background for request data.

- ZAP -> Automatic -> Prefer when testing high volumes of parameters where manual encoding is inefficient.

> ⚠️ Gap: ZAP's background encoding is not explicitly visible in the UI. If the automation fails to handle a specific character or nested format, it will result in a **server error** with no visual indication of the malformed payload being sent.

**Gotchas** **server error**: Missing encoding or incorrect request headers will trigger backend failures.

## Multi-Format Encoding and Decoding

Encountering non-URL encoding like **Base64** in cookies or requiring data conversion for backend compatibility.

### Burp Decoder and Inspector

Chain decoding/encoding by selecting new methods in the output pane

```
Decoder > Input <HASH> > Decode as > Base64
```

Modify decoded JSON content

```
{"username":"admin", "is_admin":true}
```

Use Inspector for quick inline conversion in Proxy/Repeater tabs

```
Inspector > URL/Base64/Hex
```

- Burp Decoder -> Output Pane Selection -> Prefer for complex multi-stage encoding (e.g., Base64 inside URL).
- Burp Inspector -> UI Sidebar -> Prefer for fast viewing without switching tabs.

### ZAP Encoder/Decoder/Hash

Open the encoding utility

```
CTRL+E
```

Create persistent custom views for specific engagement needs

```
Add New Tab > Select Encoders/Decoders
```

- ZAP Encoder -> `CTRL+E` -> Prefer for simultaneous viewing of multiple encoding formats for the same string.

**Edge cases**

- **Full URL-Encoding** or **Unicode URL encoding** are required when payloads contain high-density special characters that standard key-character encoding misses.

**Gotchas** **Manual copy-paste required**: ZAP does not natively chain output to input; you must manually copy the result back to the top field for sequential encoding.