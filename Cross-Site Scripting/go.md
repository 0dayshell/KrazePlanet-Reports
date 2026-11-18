## Reflected XSS - Shopify

## Description
**Affected URL:** `https://go.pwnedbox.site/`

A reflected Cross-Site Scripting (XSS) vulnerability exists in the `returnTo` parameter used by the "Products" link/functionality of the application. User input submitted through the `returnTo` URL parameter is reflected directly into the page response without any input validation or output encoding. Because the input is rendered as raw HTML in the browser, an attacker can inject arbitrary JavaScript that executes in the context of the vulnerable domain.

This is a security issue because the injected script runs with the same privileges as the legitimate site's own code. The payload is delivered through a crafted link, so any user who opens that link can be targeted.

## Steps to Reproduce
1. Open the application in a browser: `https://go.pwnedbox.site/`
2. Click on "Products", which generates a URL containing a `returnTo` parameter.
3. Modify the `returnTo` parameter value to `navi'"<>` and observe that the payload is reflected on the page.
4. Modify the `returnTo` parameter to the payload `'"<><script>alert(1)</script>`. Resulting URL: `https://go.pwnedbox.site/?returnTo=%27%22%3C%3E%3Cscript%3Ealert(1)%3C/script%3E`
5. Submit the request and observe that a JavaScript alert box with the value `1` is displayed, confirming successful reflected XSS execution.

## POC

Step 1: Click "Products" and modify the `returnTo` parameter to `navi'"<>` to confirm the input is reflected unsanitized.

<img width="489" height="116" alt="image" src="https://github.com/user-attachments/assets/a3f479cd-e057-4eae-b8fe-4ab68753cbb8" />

Step 2: Modify the `returnTo` parameter to `'"<><script>alert(1)</script>`:

<img width="956" height="335" alt="image" src="https://github.com/user-attachments/assets/d4ec0890-54fd-460d-9765-6d48dad49bcb" />

**Result:** The payload is returned unsanitized in the response and executed by the browser, displaying an `alert(1)` popup, confirming reflected XSS via the `returnTo` parameter.

## Impact
1. Attacker-controlled script executes in the victim's browser session.
2. Session cookies/tokens can be stolen if not `HttpOnly`.
3. Victim accounts can be taken over via session hijacking.
4. Fake login forms can be injected to phish credentials.
5. Victims can be redirected to malicious or attacker-controlled sites.

## Remediation
1. Apply output encoding to the `returnTo` parameter before rendering it in responses.
2. Validate the `returnTo` value against an allowlist of permitted internal paths only.
3. Avoid inserting unsanitized input into the DOM via unsafe sinks.
4. Enforce a strict CSP to block inline script execution.
5. Set `HttpOnly`, `Secure`, and `SameSite` flags on session cookies.
