# 🔏 HTTPS with win-acme

win-acme is an ACME client for Windows. Let's Encrypt is the certificate authority. win-acme creates the private key and certificate request, completes domain validation, installs the issued certificate in Windows, updates the IIS binding, and creates a renewal task.

Let's Encrypt signs the certificate; it does not proxy or carry the website's normal traffic.

## Prerequisites

- The public hostname resolves to the IIS server.
- The IIS site responds for that hostname.
- TCP 80 is reachable externally for HTTP-01 validation.
- TCP 443 is available for HTTPS.
- You have administrator access.

## 1. Download and run win-acme

1. Download the current release from [win-acme.com](https://www.win-acme.com/).
2. Extract it to a stable location such as `C:\win-acme`.
3. Right-click `wacs.exe` and select **Run as administrator**.

Do not run the executable permanently from a temporary downloads folder because the scheduled renewal depends on the installation path.

## 2. Request the certificate

The exact menu labels can vary by win-acme version.

1. Choose the option to create a new certificate.
2. Select the IIS binding/site containing `example.com`.
3. Use HTTP-01 validation.
4. Allow win-acme to install the certificate in the Windows certificate store.
5. Allow it to create or update the HTTPS binding in IIS.
6. Allow creation of the scheduled renewal task.

During HTTP-01 validation, Let's Encrypt requests a temporary challenge file through the public hostname. DNS, port 80, IIS routing, and firewall rules must all allow that request to reach the correct site.

## 3. Verify the IIS binding

Open **IIS Manager > Sites > MySite > Bindings** and confirm:

| Type | Port | Hostname | Certificate |
|---|---:|---|---|
| HTTP | 80 | example.com | None |
| HTTPS | 443 | example.com | Let's Encrypt certificate |

When several HTTPS sites share one address, use the appropriate hostname and SNI configuration.

## 4. Test HTTPS

```powershell
Test-NetConnection example.com -Port 443
Invoke-WebRequest https://example.com -UseBasicParsing
```

In a browser, confirm:

- The URL uses `https://`.
- No certificate warning appears.
- The certificate covers the requested hostname.
- The issuer chain is trusted.
- The validity dates are current.

## 5. Verify renewal

Open **Task Scheduler > Task Scheduler Library** and locate the win-acme renewal task. Check its status, history, last result, and next run time.

Use win-acme's built-in renewal test/dry-run option for the installed version. Do not wait until certificate expiry to discover that validation is broken.

Let's Encrypt certificates are short-lived, so working automatic renewal is part of the HTTPS deployment—not an optional extra.

## Renewal dependencies

Renewal can fail later if:

- DNS is changed.
- Port 80 becomes unreachable for HTTP-01.
- The IIS site or binding is removed.
- the win-acme directory is moved.
- The scheduled task loses permission.
- A proxy or firewall blocks the challenge path.
