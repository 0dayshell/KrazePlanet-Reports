## Reflected XSS - Case-Insensitive Filter Bypass

## Description
**Affected URL:** `https://directory.pwnedbox.site/`

A reflected Cross-Site Scripting (XSS) vulnerability exists in the application's input form. User input submitted through the `fname` and `lname` URL parameters is reflected into the page response. The application attempts to filter the literal `<script>` tag, but this blacklist-based filter is case-sensitive and does not account for mixed-case variations, allowing the filter to be bypassed.

This is a security issue because the injected script runs with the same privileges as the legitimate site's own code, and a case-based filter bypass shows the sanitization is insufficient. The payload is delivered through a crafted link, so any user who opens that link can be targeted.

## Steps to Reproduce
1. Open the application in a browser: `https://directory.pwnedbox.site/`
2. Locate the First Name and Last Name fields.
3. Enter the payload `navi` into the fields and submit. Observe that the payload is reflected on the page, confirming no character-level filtering on plain text.
4. Modify the payload to `'"<><sCriPt>alert(1)</sCriPt>`, using mixed case to bypass the case-sensitive `script` filter, and submit.
6. Observe that a JavaScript alert box with the value `1` is displayed, confirming successful reflected XSS execution via the filter bypass.

## POC

Step 1: Enter `navi` into the fields and submit to confirm the input is reflected.

<img width="959" height="502" alt="image" src="https://github.com/user-attachments/assets/972cf1a0-9b4f-4948-9a00-f7a8c332687e" />

Step 2: Enter `'"<><sCriPt>alert(1)</sCriPt>` and submit.

<img width="959" height="500" alt="image" src="https://github.com/user-attachments/assets/ed522012-6293-4cec-abc7-44ca7e62dd0b" />

**Result:** The case-sensitive filter does not strip `sCriPt`, and the payload executes, displaying an `alert(1)` popup, confirming reflected XSS via the `fname`/`lname` parameters.

## Impact
1. Attacker-controlled script executes in the victim's browser session.
2. Session cookies/tokens can be stolen if not `HttpOnly`.
3. Victim accounts can be taken over via session hijacking.
4. Fake login forms can be injected to phish credentials.
5. Victims can be redirected to malicious or attacker-controlled sites.

## Remediation
1. Do not rely on blacklisting the literal `script` string; use proper output encoding instead.
2. Apply output encoding to the `fname` and `lname` parameters before rendering them in responses.
3. Validate input against an allowlist of expected characters on the server side.
4. Enforce a strict CSP to block inline script execution regardless of filter bypasses.
5. Set `HttpOnly`, `Secure`, and `SameSite` flags on session cookies.



