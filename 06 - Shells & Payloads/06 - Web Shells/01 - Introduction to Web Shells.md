## METHODOLOGY

1. External perimeter hardened with no exposed services (SMB). Prioritize web application attack surface.
2. Map upload vectors: public forms, authenticated profile settings, or self-registration portals.
3. Identify server technology: Tomcat, Axis2, and WebLogic allow deployment via WAR files.
4. Check for misconfigured FTP services allowing direct writes to the webroot.
5. Upload payload matching the server language (PHP, JSP, ASP.NET).
6. Upgrade browser-based web shell to an interactive reverse shell for stability.

---

## FOOTHOLD VIA FILE UPLOAD

Target lacks common vulnerable services (SMB) and requires web-based entry via unrestricted upload or misconfiguration.

### Web Shell Deployment

Uploaded payload gives remote code execution capability within the browser.

> ⚠️ Gap: Source lacks specific web shell syntax for PHP, JSP, and ASP.NET payloads, which will result in no execution if generic files are uploaded.

### Self-Registration and Profile Uploads

Authenticated or self-registered sessions providing profile picture uploads can be abused to bypass client-side checks.

> ⚠️ Gap: Source provides no methodology for bypassing client-side checks, which will cause the upload to fail against modern browsers/apps.

## JSP WAR DEPLOYMENT

Management interfaces for specific application servers allow for JSP execution.

### WAR File Upload

1. Identify Tomcat, Axis2, or WebLogic.
2. Deploy JSP code packaged as a WAR file using native application functionality.

> ⚠️ Gap: Source lacks the command-line or GUI steps to package a WAR file or interact with management interfaces, causing the technique to fail at the delivery stage.

## FTP WEBROOT MISCONFIGURATION

Direct operating system interaction via a web shell when file system permissions are weak.

### Direct Webroot Write

1. Access FTP service.
2. Upload web shell payload directly into the server's webroot.

**Application-driven file deletion** **Payloads may be deleted by the application after a specific time interval**, leading to loss of access before a reverse shell is established.

## WEB SHELL INTERACTION

Browser-based interaction with the underlying operating system.

**Unstable session** **Relying on the web shell alone is unstable and unreliable**; persistence requires upgrading to an interactive reverse shell.