## Reflected XSS - Script & Img Tag Filter

## Description
**Affected URL:** `https://support.pwnedbox.site/`

A reflected Cross-Site Scripting (XSS) vulnerability exists in the application's input form. User input submitted through the `fname` and `lname` URL parameters is reflected directly into the page response without any input validation or output encoding. Because the input is rendered as raw HTML in the browser, an attacker can inject arbitrary JavaScript that executes in the context of the vulnerable domain.

This is a security issue because the injected script runs with the same privileges as the legitimate site's own code. The payload is delivered through a crafted link, so any user who opens that link can be targeted.

## Steps to Reproduce
1. Open the application in a browser: `https://support.pwnedbox.site/`
2. Locate the First Name and Last Name fields.
3. Enter the payload `navi'"<>` into both fields and submit. Observe that the payload is reflected on the page, confirming no input sanitization.
4. Modify the payload in the First Name field to `'"<><sCriPt>alert(1)</sCriPt>` and the Last Name field to `'"<><sCriPt>alert(1)</sCriPt>`.
5. Submit the request and observe that JavaScript alert boxes with the value `1` are displayed for both fields, confirming successful reflected XSS execution.

## POC

Step 1: Enter `navi'"<>` into the First Name and Last Name fields and submit to confirm the input is reflected unsanitized.

<img width="956" height="476" alt="image" src="https://github.com/user-attachments/assets/1b335cc9-e746-4028-87c5-81d0f939c8fc" />

Step 2: Submit the payload via the first name and last name field

<img width="957" height="277" alt="image" src="https://github.com/user-attachments/assets/98b28100-edaf-4d55-bb8a-c78be64cdb24" />

**Result:** The payloads are returned unsanitized in the response and executed by the browser, displaying `alert(1)` popups from both the `fname` and `lname` parameters, confirming reflected XSS.

## Impact
1. Attacker-controlled script executes in the victim's browser session.
2. Session cookies/tokens can be stolen if not `HttpOnly`.
3. Victim accounts can be taken over via session hijacking.
4. Fake login forms can be injected to phish credentials.
5. Victims can be redirected to malicious or attacker-controlled sites.

## Remediation
1. Apply output encoding to the `fname` and `lname` parameters before rendering them in responses.
2. Validate input against an allowlist of expected characters and length on the server side.
3. Avoid inserting unsanitized input into the DOM via unsafe sinks.
4. Enforce a strict CSP to block inline script execution.
5. Set `HttpOnly`, `Secure`, and `SameSite` flags on session cookies.
