# 🌐 IIS Website Setup

## Prerequisites

- A supported Windows workstation or server
- Local administrator access
- A registered domain
- Public DNS pointing to the server
- Internet access
- Inbound TCP 80 and 443 permitted
- Website files, including a default document such as `index.html`

Example values used in this repository:

```text
Hostname: example.com
Website folder: C:\inetpub\wwwroot\mysite
HTTP port: 80
HTTPS port: 443
```

## 1. Verify DNS

From PowerShell:

```powershell
Resolve-DnsName example.com
# Alternative
nslookup example.com
```

Confirm the returned address is the intended public address. DNS must be correct before ACME HTTP-01 validation can succeed.

## 2. Install IIS

1. Search Windows for **Turn Windows features on or off**.
2. Select **Internet Information Services**.
3. Under **Web Management Tools**, enable **IIS Management Console**.
4. Under **World Wide Web Services > Common HTTP Features**, enable at least:
   - Default Document
   - HTTP Errors
   - Static Content
5. Apply the changes.
6. Browse to `http://localhost`.

If the IIS landing page loads, the service is installed and responding.

On Windows Server, the equivalent role can be installed with Server Manager or PowerShell:

```powershell
Install-WindowsFeature Web-Server -IncludeManagementTools
```

## 3. Create the website folder

```powershell
New-Item -ItemType Directory -Path "C:\inetpub\wwwroot\mysite"
```

Copy the website files into the folder. Ensure the application-pool identity has only the permissions it needs; do not grant broad write permission unless the application requires it.

## 4. Create the IIS site

1. Open **Internet Information Services (IIS) Manager**.
2. Right-click **Sites**, then choose **Add Website**.
3. Set:
   - Site name: `MySite`
   - Physical path: `C:\inetpub\wwwroot\mysite`
   - Type: `http`
   - IP address: `All Unassigned` or the intended server address
   - Port: `80`
   - Host name: `example.com`
4. Select **Start Website immediately**.
5. Click **OK**.

If several sites share the same IP address, the hostname distinguishes them.

## 5. Confirm the default document

Open **Default Document** for the site and confirm that the actual entry file—such as `index.html`—is present and ordered appropriately.

## 6. Test HTTP

```powershell
Test-NetConnection example.com -Port 80
Invoke-WebRequest http://example.com -UseBasicParsing
```

Complete this HTTP test before requesting a certificate.
