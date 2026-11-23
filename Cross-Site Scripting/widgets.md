## Reflected XSS - Reddit

## Description
**Affected URL:** `https://widgets.pwnedbox.site/`

A reflected Cross-Site Scripting (XSS) vulnerability exists in a hidden `post_id` parameter of the application, discovered through parameter enumeration. User input submitted through the `post_id` URL parameter is reflected directly into the page response without any input validation or output encoding, allowing a crafted script payload to execute directly in the browser.

This is a security issue because an undocumented parameter is exploitable without any visible form exposing it, and the injected script runs with the same privileges as the legitimate site's own code.

## Steps to Reproduce
1. Open the application in a browser: `https://widgets.pwnedbox.site/`
2. Perform parameter enumeration and discover the undocumented `post_id` parameter.
3. Submit the test payload `navi'"<>` via the `post_id` parameter: `https://widgets.pwnedbox.site/?post_id=navi'"<>` and inspect the source code to confirm it is reflected unsanitized.
4. Modify the `post_id` parameter to the payload `navi'"<><ScRiPt>\u0063onfirm("navi")</ScRiPt>`.
5. Submit the request and observe that a `confirm("navi")` dialog is displayed, confirming successful reflected XSS execution via the `post_id` parameter.

## POC

Step 1: Submit `navi'"<>` via the discovered `post_id` parameter and inspect the source code to confirm it is reflected unsanitized.

<img width="959" height="353" alt="image" src="https://github.com/user-attachments/assets/964cd319-c50e-4af0-97f0-414f9259c75d" />

Step 2: Submit the payload via the `post_id` parameter: navi'"<><ScRiPt>\u0063onfirm("navi")</ScRiPt>

<img width="951" height="340" alt="image" src="https://github.com/user-attachments/assets/57ef9287-3c2a-4e7c-b3fb-71c0ed7862b2" />

**Result:** The payload is returned unsanitized in the response and executed by the browser, displaying a `confirm("navi")` dialog, confirming reflected XSS via the `post_id` parameter.

## Impact
1. Attacker-controlled script executes in the victim's browser session.
2. Session cookies/tokens can be stolen if not `HttpOnly`.
3. Victim accounts can be taken over via session hijacking.
4. Fake login forms can be injected to phish credentials.
5. Victims can be redirected to malicious or attacker-controlled sites.

## Remediation
1. Apply output encoding to the `post_id` parameter before rendering it in responses.
2. Validate the `post_id` value against an allowlist of expected formats (e.g., numeric IDs only).
3. Avoid inserting unsanitized input into the DOM via unsafe sinks.
4. Enforce a strict CSP to block inline script execution.
5. Set `HttpOnly`, `Secure`, and `SameSite` flags on session cookies.
