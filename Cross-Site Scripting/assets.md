## Reflected XSS - PUBG

## Description
**Affected URL:** `https://assets.pwnedbox.site/`

A reflected Cross-Site Scripting (XSS) vulnerability exists in the search functionality of the application. User input submitted through the search parameter is reflected directly into the page response without any input validation or output encoding. Because the input is rendered as raw HTML in the browser, an attacker can inject arbitrary JavaScript that executes in the context of the vulnerable domain.

This is a security issue because the injected script runs with the same privileges as the legitimate site's own code. The payload is delivered through a crafted link, so any user who opens that link can be targeted.

## Steps to Reproduce
1. Open the application in a browser: `https://assets.pwnedbox.site/`
2. Locate the search field.
3. Enter the payload `navi'"<>` and submit the search.
4. Inspect the page source and observe that the payload is reflected unencoded, confirming no input sanitization.
5. Modify the payload to `'"<><script>alert(1)</script>` and submit.
6. Observe that a JavaScript alert box with the value `1` is displayed, confirming successful reflected XSS execution.

## POC

Step 1: Enter the payload `navi'"<>` into the search field and submit. Inspect the source to confirm the payload is reflected without encoding.

<img width="522" height="113" alt="image" src="https://github.com/user-attachments/assets/474f1020-41de-4f41-9563-3e63afd527a0" />

Step 2: Modify the payload to `'"<><script>alert(1)</script>` and submit.

<img width="958" height="533" alt="image" src="https://github.com/user-attachments/assets/25f9a0af-96be-44ca-ba2f-b6d964077b8c" />

**Result:** The payload is returned unsanitized in the response and executed by the browser, displaying an `alert(1)` popup, confirming reflected XSS.

## Impact
1. Attacker-controlled script executes in the victim's browser session.
2. Session cookies/tokens can be stolen if not `HttpOnly`.
3. Victim accounts can be taken over via session hijacking.
4. Fake login forms can be injected to phish credentials.
5. Victims can be redirected to malicious or attacker-controlled sites.

## Remediation
1. Apply output encoding to all user-controlled data before rendering it in responses.
2. Validate input against an allowlist of expected characters and length.
3. Avoid inserting unsanitized input into the DOM via unsafe sinks.
4. Enforce a strict CSP to block inline script execution.
5. Set `HttpOnly`, `Secure`, and `SameSite` flags on session cookies.

