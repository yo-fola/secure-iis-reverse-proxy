# 🔁 HTTPS Redirect and HSTS

Configure redirection first, confirm HTTPS is reliable, and only then enable HSTS.

## Install URL Rewrite

Install Microsoft's [IIS URL Rewrite Module](https://www.iis.net/downloads/microsoft/url-rewrite), then reopen IIS Manager if the feature does not appear.

## Create the HTTP-to-HTTPS rule

At the website level:

1. Open **URL Rewrite**.
2. Select **Add Rule(s) > Blank rule** under Inbound Rules.
3. Name the rule `Redirect HTTP to HTTPS`.
4. Configure the match:
   - Requested URL: Matches the Pattern
   - Using: Regular Expressions
   - Pattern: `(.*)`
   - Ignore case: enabled
5. Add a condition:
   - Condition input: `{HTTPS}`
   - Check if input string: Matches the Pattern
   - Pattern: `^OFF$`
   - Ignore case: enabled
6. Configure the action:
   - Action type: Redirect
   - Redirect URL: `https://{HTTP_HOST}/{R:1}`
   - Append query string: enabled
   - Redirect type: Permanent (301)
7. Apply the rule.

The equivalent configuration is included in [the example web.config](../examples/web.config.example).

## Verify the redirect

```powershell
$response = Invoke-WebRequest http://example.com/test?source=check -MaximumRedirection 0 -SkipHttpErrorCheck
$response.StatusCode
$response.Headers.Location
```

Expected:

```text
301
https://example.com/test?source=check
```

## Enable HSTS

HSTS tells supporting browsers to use HTTPS for future requests. Add this only after the site and renewal process have been confirmed.

In **HTTP Response Headers**, add:

```text
Name: Strict-Transport-Security
Value: max-age=31536000; includeSubDomains
```

Or define it under `system.webServer/httpProtocol/customHeaders` in `web.config`.

### Important cautions

- `31536000` seconds is one year.
- `includeSubDomains` applies the policy to every subdomain. Remove it unless all subdomains support HTTPS.
- Do not add `preload` casually. Browser preload lists are difficult to reverse.
- HSTS is remembered by the browser, so a broken certificate can make the site inaccessible instead of allowing a bypass.

## Verify the header

```powershell
(Invoke-WebRequest https://example.com -UseBasicParsing).Headers["Strict-Transport-Security"]
```

Expected:

```text
max-age=31536000; includeSubDomains
```
