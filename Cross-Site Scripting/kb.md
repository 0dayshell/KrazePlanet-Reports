## Reflected XSS - Search Filter Bypass

## Description
**Affected URL:** `https://kb.pwnedbox.site/`

A reflected Cross-Site Scripting (XSS) vulnerability exists in the search functionality of the application. User input submitted through the search bar is reflected directly into the page response without any input validation or output encoding, allowing a crafted script payload to execute directly in the browser.

This is a security issue because the injected script runs with the same privileges as the legitimate site's own code. The payload is delivered through a crafted link, so any user who opens that link can be targeted.

## Steps to Reproduce
1. Open the application in a browser: `https://kb.pwnedbox.site/`
2. Locate the search bar.
3. Enter the payload `navi'"<>` and submit the search. Observe that the payload is reflected on the page, confirming no input sanitization.
4. Modify the payload to `navi'"<><ScRiPt>\u0063onfirm("navi")</ScRiPt>` and submit.
5. Observe that a `confirm("navi")` dialog is displayed, confirming successful reflected XSS execution.

## POC

Step 1: Enter `navi'"<>` into the search bar and submit to confirm the input is reflected unsanitized.

<img width="935" height="488" alt="image" src="https://github.com/user-attachments/assets/b50f792b-9e5b-4a29-94da-324157f8abd6" />

Step 2: Submit the payload `navi'"<><ScRiPt>\u0063onfirm("navi")</ScRiPt>` via the search bar.

<img width="956" height="452" alt="image" src="https://github.com/user-attachments/assets/59c83752-a729-40e7-9bf1-ad655af3b972" />

**Result:** The payload is returned unsanitized in the response and executed by the browser, displaying a `confirm("navi")` dialog, confirming reflected XSS via the search parameter.

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
