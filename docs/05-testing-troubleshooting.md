# 🧪 Testing and Troubleshooting

Test each layer separately. A browser error alone does not identify whether the fault is DNS, firewall, IIS, TLS, rewriting, ARR, or the backend.

## Validation checklist

```powershell
Resolve-DnsName example.com
Test-NetConnection example.com -Port 80
Test-NetConnection example.com -Port 443

# Run from a PowerShell version that supports these parameters.
Invoke-WebRequest http://example.com -MaximumRedirection 0 -SkipHttpErrorCheck
Invoke-WebRequest https://example.com -UseBasicParsing

# Test the example backend locally.
Test-NetConnection 127.0.0.1 -Port 8080
Invoke-WebRequest http://127.0.0.1:8080/ -UseBasicParsing
```

Also verify:

- IIS site state is **Started**.
- The hostname is on the correct binding.
- The correct certificate is selected on port 443.
- The win-acme scheduled task is healthy.
- HTTP returns 301 and preserves the path/query string.
- HTTPS returns the HSTS header after it has been enabled.
- The upstream is not reachable from the public internet.

## Common symptoms

| Symptom | Likely checks |
|---|---|
| Domain does not resolve | DNS record, propagation, authoritative name servers |
| Connection timeout | Edge firewall/NAT, Windows Firewall, ISP filtering, service listener |
| IIS default page appears | Hostname binding or site selection |
| 404 during ACME validation | Site binding, challenge path, rewrite exclusions, proxy rules |
| Certificate name mismatch | HTTPS binding, hostname/SNI, selected certificate |
| Redirect loop | Proxy awareness of HTTPS, forwarded scheme, conflicting redirect rules |
| 502.3 Bad Gateway | Backend state, address, port, protocol, firewall, timeout |
| HSTS blocks access | Repair HTTPS; browsers will not permit an insecure fallback |
| Renewal fails | DNS, port 80, scheduled-task permission, moved win-acme directory |

## Logs and diagnostics

- IIS access logs: `C:\inetpub\logs\LogFiles`
- Windows Event Viewer: Application and System logs
- win-acme logs: check the configured win-acme log path
- Failed Request Tracing: enable temporarily for detailed IIS diagnostics
- Backend logs: inspect application startup and request errors
- `netstat -ano`: confirm which processes own expected listening ports

Example:

```powershell
Get-NetTCPConnection -State Listen |
  Sort-Object LocalPort |
  Select-Object LocalAddress, LocalPort, OwningProcess
```

## ACME rewrite consideration

HTTP-01 validation must reach the challenge resource. If a broad redirect or proxy rule interferes with `/.well-known/acme-challenge/`, use win-acme's IIS integration and verify the generated validation route. Avoid making untested exclusions during a live renewal window.

## Safe rollback

Before major IIS changes:

```powershell
& "$env:windir\system32\inetsrv\appcmd.exe" add backup "before-secure-proxy-change"
```

List and restore backups with `appcmd list backup` and `appcmd restore backup <name>`. Restoring changes the IIS configuration, so use it only when the target backup is confirmed.
