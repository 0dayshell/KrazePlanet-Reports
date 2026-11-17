## Reflected XSS - Page Heading

## Description
**Affected URL:** `https://docs.pwnedbox.site/`

A reflected Cross-Site Scripting (XSS) vulnerability exists in the GET Parameter Reflection feature of the application. The visible form submits input through the `fname` and `lname` parameters, which appear to be sanitized/validated server-side. However, a hidden `q` parameter also exists on the same endpoint and is reflected into the page without any validation or output encoding, allowing a crafted script payload to execute directly in the browser.

This is a security issue because an undocumented parameter bypasses the input validation applied to the visible form fields, and the injected script runs with the same privileges as the legitimate site's own code.

## Steps to Reproduce
1. Open the application in a browser: `https://docs.pwnedbox.site/`
2. Enter the payload `navi'"<>` into the First Name and Last Name fields and submit. The URL becomes `https://docs.pwnedbox.site/?fname=navi%27%22%3C%3E&lname=navi%27%22%3C%3E`, and the payload is reflected but appears to be validated/filtered in the page source.
3. Remove the `fname` and `lname` parameters from the URL and instead add a `q` parameter directly: `https://docs.pwnedbox.site/?q=navi'"<>`.
4. Observe that the `q` parameter value is reflected on the page without any filtering.
5. Modify the `q` parameter to a mixed-case script payload to avoid any case-sensitive filter: `https://docs.pwnedbox.site/?q=navi'"<><sCRiPt>alert("HI")</sCrIpt>`
6. Submit the request and observe that a JavaScript alert box with the value `HI` is displayed, confirming successful reflected XSS execution through the hidden `q` parameter.

## POC

Step 1: Submit `navi'"<>` through the visible form fields to confirm the `fname`/`lname` parameters are validated/filtered.

<img width="956" height="447" alt="image" src="https://github.com/user-attachments/assets/929d27bf-da31-4c4f-8be1-024f53961e88" />

<img width="794" height="197" alt="image" src="https://github.com/user-attachments/assets/32a619bb-9811-426a-832c-49c4c73f6a35" />


Step 2: Remove `fname`/`lname` and instead pass the payload directly via the hidden `q` parameter: `https://docs.pwnedbox.site/?q=navi'"<><sCRiPt>alert("HI")</sCrIpt>`

<img width="1500" height="538" alt="image" src="https://github.com/user-attachments/assets/d7e3096b-28a3-4678-bc0a-8e2913ea5f5e" />


**Result:** The `q` parameter is reflected without sanitization, and the mixed-case `<sCRiPt>` tag executes, displaying an `alert("HI")` popup, confirming reflected XSS.

## Impact
1. Attacker-controlled script executes in the victim's browser session.
2. Session cookies/tokens can be stolen if not `HttpOnly`.
3. Victim accounts can be taken over via session hijacking.
4. Fake login forms can be injected to phish credentials.
5. Victims can be redirected to malicious or attacker-controlled sites.

## Remediation
1. Apply output encoding to the `q` parameter wherever it is reflected.
2. Validate and sanitize all parameters, not just the ones tied to visible form fields.
3. Remove or disable undocumented/hidden parameters that aren't needed.
4. Enforce a strict CSP to block inline script execution.
5. Set `HttpOnly`, `Secure`, and `SameSite` flags on session cookies.
