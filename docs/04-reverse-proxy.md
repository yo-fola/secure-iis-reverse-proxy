# 🔀 ARR Reverse Proxy

This section documents the reusable IIS reverse-proxy pattern. The attached implementation notes did not record the final upstream hostname or port, so this guide deliberately uses `http://127.0.0.1:8080` as a placeholder.

## Design goal

Expose only IIS on TCP 443 while the application listens on localhost or a private address. IIS terminates public TLS and forwards matching requests to the backend.

## Prerequisites

- The IIS website already works over HTTPS.
- IIS URL Rewrite is installed.
- Application Request Routing (ARR) is installed.
- The backend responds locally.
- The backend port is blocked from public access.

Test the backend before configuring IIS:

```powershell
Test-NetConnection 127.0.0.1 -Port 8080
Invoke-WebRequest http://127.0.0.1:8080/ -UseBasicParsing
```

## 1. Enable ARR proxying

1. Open IIS Manager and select the **server** node.
2. Open **Application Request Routing Cache**.
3. Select **Server Proxy Settings**.
4. Enable **Proxy**.
5. Apply the change.

Proxy enablement is server-wide; routing rules can still be scoped to a specific website.

## 2. Create the inbound proxy rule

At the website level:

1. Open **URL Rewrite**.
2. Add a **Blank rule** under Inbound Rules.
3. Name it `Reverse Proxy to Backend`.
4. Match `(.*)`, or a narrower route such as `^api/(.*)`.
5. Set the action type to **Rewrite**.
6. Use an upstream URL such as:
   - Whole site: `http://127.0.0.1:8080/{R:1}`
   - API only: `http://127.0.0.1:8080/{R:1}`
7. Enable **Append query string**.
8. Enable **Stop processing of subsequent rules**.
9. Apply and restart the website if required.

The HTTPS redirect rule should appear before the proxy rule so insecure requests are redirected before proxying.

## 3. Protect the backend

- Prefer binding the backend to `127.0.0.1`.
- If a private interface is required, allow the backend port only from the IIS server.
- Do not create a public NAT/firewall rule for the upstream port.
- Authenticate and authorize requests in the application; TLS termination does not replace application security.
- Configure WebSocket support only if the application needs it.
- Set explicit request-size and timeout limits appropriate to the workload.

## 4. Forwarded information

Many applications need the original scheme, host, and client address. Confirm how the backend framework handles:

```text
X-Forwarded-For
X-Forwarded-Proto
Host
```

Trust forwarded headers only when requests arrive from the known IIS proxy. Do not blindly trust client-supplied forwarded headers.

## 5. Test the complete path

```powershell
Invoke-WebRequest https://example.com/ -UseBasicParsing
```

Then compare:

- Direct backend response
- IIS-proxied response
- IIS access logs
- Backend application logs

A `502.3 Bad Gateway` usually means IIS cannot connect to the upstream, the port/protocol is wrong, or the backend timed out.

## 6. Optional TLS to the backend

TLS between IIS and the backend can be justified when traffic crosses machines or an untrusted network. For a same-host loopback connection, local HTTP may be acceptable if the host itself is secured. If HTTPS is used upstream, validate the backend certificate rather than disabling certificate checks.
