## Pre-configured Proxy Browser

Interception is required immediately without performing manual browser or certificate configuration.

Launch Burp's embedded browser with integrated proxy routing

```
Proxy > Intercept > Open Browser
```

Launch ZAP's pre-configured Firefox instance from the toolbar

```
Quick Start > Firefox Icon
```

- Tool comparison
    - Burp Suite -> `Open Browser` -> prefer for Chromium-based internal environments.
    - ZAP -> Firefox Icon -> prefer for Firefox-based internal environments.

## External Proxy Configuration

Analysis requires a standalone browser or the default port 8080 is already bound by another service.

Modify Burp listener port to avoid conflicts

```
Proxy > Proxy settings > Proxy listeners
```

Modify ZAP listener port to avoid conflicts

```
Tools > Options > Network > Local Servers/Proxies
```

Add a new proxy profile to FoxyProxy for quick switching

```
IP: 127.0.0.1
Port: <PORT>
```

- Gotchas
    - **Port collision** prevents the proxy service from starting if the listener port is already in use.

## CA Certificate Installation

HTTPS traffic triggers security warnings or fails to route through the proxy in an external browser.

Download the PortSwigger CA certificate while proxied

```
http://burp
```

Export the ZAP CA certificate to the local filesystem

```
Tools > Options > Network > Server Certificates > Save
```

Access the Firefox certificate manager for manual import

```
about:preferences#privacy > View Certificates > Authorities > Import
```

- Dangerous / misconfigured settings
    
    - Disabling **Trust this CA to identify websites** during certificate import.
    - Disabling **Trust this CA to identify email users** during certificate import.
- Gotchas
    
    - **HTTPS handshake failure** occurs if the CA certificate is not explicitly trusted for website identification after import.