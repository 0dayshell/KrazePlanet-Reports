## Reflected XSS - Imgur Mobile

## Description
**Affected URL:** `https://media.pwnedbox.site/index.php/account/`

A reflected Cross-Site Scripting (XSS) vulnerability exists in the messaging/account feature of the application. A user-controlled value supplied as a URL path segment (between `/account/` and `/messages`) is reflected directly into the page response without any input validation or output encoding. Because the input is rendered as raw HTML in the browser, an attacker can inject arbitrary JavaScript that executes in the context of the vulnerable domain.

This is a security issue because the injected script runs with the same privileges as the legitimate site's own code. The payload is delivered through a crafted link, so any user who opens that link can be targeted.

## Steps to Reproduce
1. Open the application in a browser: `https://media.pwnedbox.site/`
2. Click on a conversation/message, which loads a URL in the form `https://media.pwnedbox.site/index.php/account/<username>/messages`.
3. Replace `<username>` with the test payload `navi'"<>` and observe that it is reflected on the page, confirming no sanitization.
4. Replace `<username>` with the payload `'"<><script>alert(1)</script>`.
5. Submit the request and observe that a JavaScript alert box with the value `1` is displayed, confirming successful reflected XSS execution via the URL path segment.

## POC

Step 1: Replace the account value in the URL path with `navi'"<>` to confirm the input is reflected unsanitized.

<img width="956" height="380" alt="image" src="https://github.com/user-attachments/assets/a17f0ffd-90fb-4088-8558-ecd755fc5cab" />

Step 2: Replace the account value with `'"<><script>alert(1)</script>`:

<img width="956" height="413" alt="image" src="https://github.com/user-attachments/assets/d849e6e9-8fad-4e52-92b6-032c5869430e" />

**Result:** The payload is returned unsanitized in the response and executed by the browser, displaying an `alert(1)` popup, confirming reflected XSS via the URL path parameter.

## Impact
1. Attacker-controlled script executes in the victim's browser session.
2. Session cookies/tokens can be stolen if not `HttpOnly`.
3. Victim accounts can be taken over via session hijacking.
4. Fake login forms can be injected to phish credentials.
5. Victims can be redirected to malicious or attacker-controlled sites.

## Remediation
1. Apply output encoding to the path-based account value before rendering it in responses.
2. Validate the value against an allowlist of expected characters (e.g., alphanumeric usernames/IDs only).
3. Avoid inserting unsanitized path input into the DOM via unsafe sinks.
4. Enforce a strict CSP to block inline script execution.
5. Set `HttpOnly`, `Secure`, and `SameSite` flags on session cookies.
