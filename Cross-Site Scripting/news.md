## Reflected XSS - HTML Tag Filter Bypass

## Description
**Affected URL:** `https://news.pwnedbox.site/`

A reflected Cross-Site Scripting (XSS) vulnerability exists in a hidden `title` parameter of the application. The visible `fname` and `lname` parameters are filtered/input-validated, but parameter enumeration revealed an undocumented `title` parameter on the same endpoint, which reflects user input into the page response without any validation or output encoding. Because the input is rendered as raw HTML, an attacker can inject a crafted payload that executes in the browser.

This is a security issue because an undocumented parameter bypasses the input validation applied to the visible form fields, and the injected payload runs with the same privileges as the legitimate site's own code.

## Steps to Reproduce
1. Open the application in a browser: `https://news.pwnedbox.site/`
2. Enter the payload `navi'"<>` into the `fname` and `lname` fields and submit. Observe that the characters are filtered/sanitized in the response.
3. Through parameter enumeration, discover an undocumented `title` parameter on the same endpoint: `https://news.pwnedbox.site/?title=`
4. Submit the test payload `navi'"<>` via the `title` parameter and observe that it is reflected on the page without filtering.
5. Submit the following payload via the `title` parameter.
6. Observe that the payload breaks out of multiple HTML contexts and the `Details`/`OnToggle` event handler fires `confirm(1)`, confirming successful reflected XSS execution via the hidden `title` parameter.

## POC

Step 1: Submit `navi'"<>` via `fname`/`lname` to confirm those fields are filtered.

<img width="957" height="500" alt="image" src="https://github.com/user-attachments/assets/3ff4042b-1850-4afa-87fe-2d1cec3efd87" />

Step 2: Submit `navi'"<>` via the discovered `title` parameter to confirm it is reflected unsanitized.

<img width="956" height="526" alt="image" src="https://github.com/user-attachments/assets/1604b229-0c41-4389-8cdc-611ab6b09610" />

<img width="569" height="233" alt="image" src="https://github.com/user-attachments/assets/2598cb73-47e5-47c1-8f10-030150754921" />

Step 3: Submit the payload via the `title` parameter

<img width="959" height="275" alt="image" src="https://github.com/user-attachments/assets/e10445f9-10a0-4e84-a789-1ada21f71d24" />


**Result:** The `title` parameter is reflected without sanitization, and the `<details open ontoggle>` event handler executes automatically, triggering a `confirm(1)` dialog and confirming reflected XSS.

## Impact
1. Attacker-controlled script executes in the victim's browser session.
2. Session cookies/tokens can be stolen if not `HttpOnly`.
3. Victim accounts can be taken over via session hijacking.
4. Fake login forms can be injected to phish credentials.
5. Victims can be redirected to malicious or attacker-controlled sites.

## Remediation
1. Apply output encoding to the `title` parameter wherever it is reflected.
2. Validate and sanitize all parameters consistently, not just the ones tied to visible form fields.
3. Remove or disable undocumented/hidden parameters that aren't needed.
4. Enforce a strict CSP to block inline script execution and auto-firing event handlers.
5. Set `HttpOnly`, `Secure`, and `SameSite` flags on session cookies.
