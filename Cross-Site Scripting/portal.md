## Reflected XSS - String Concatenation Bypass

## Description
**Affected URL:** `https://portal.pwnedbox.site/`

A reflected Cross-Site Scripting (XSS) vulnerability exists in a hidden `cat` parameter of the application. The visible `fname` and `lname` parameters are filtered/input-validated, but parameter enumeration revealed additional undocumented parameters on the same endpoint, including `cat`, `id`, `number`, `page`, and `page.id`. The `cat` parameter reflects user input into the page response without any validation, and a JavaScript Unicode escape sequence bypasses the keyword-based filter blocking `confirm`.

This is a security issue because an undocumented parameter bypasses the input validation applied to the visible form fields, and the injected script runs with the same privileges as the legitimate site's own code.

## Steps to Reproduce
1. Open the application in a browser: `https://portal.pwnedbox.site/`
2. Enter the payload `navi'"<>` into the `fname` and `lname` fields and submit. Observe that the characters are filtered/sanitized in the response.
3. Perform parameter enumeration and discover additional undocumented parameters on the same endpoint: `cat`, `id`, `number`, `page`, `page.id`.
4. Submit the test payload `navi'"<>` via the `cat` parameter: `https://portal.pwnedbox.site/?cat=navi'"<>` and observe that it is reflected on the page without filtering.
5. Replace `confirm` with its JavaScript Unicode escape sequence equivalent to bypass the keyword filter: `navi35'"<><ScRiPt>\u0063onfirm("navi")</ScRiPt>`.
6. Submit the request and observe that a `confirm("navi")` dialog is displayed, confirming successful reflected XSS execution via the Unicode escape filter bypass.

## POC

Step 1: Submit `navi'"<>` via the discovered `cat` parameter to confirm it is reflected unsanitized.

<img width="959" height="257" alt="image" src="https://github.com/user-attachments/assets/405daaac-8acf-417b-ac7e-28920bbdb433" />

Step 2: Submit the payload via the `cat` parameter: navi'"<><ScRiPt>\u0063onfirm("navi35")</ScRiPt>

<img width="959" height="319" alt="image" src="https://github.com/user-attachments/assets/7a8b36c5-9ddf-46c5-acc5-b484644e66d9" />

**Result:** The `confirm` keyword filter does not account for the JavaScript Unicode escape sequence `\u0063onfirm`, and the payload executes, displaying a `confirm("navi")` dialog, confirming reflected XSS via the `cat` parameter.

## Impact
1. Attacker-controlled script executes in the victim's browser session.
2. Session cookies/tokens can be stolen if not `HttpOnly`.
3. Victim accounts can be taken over via session hijacking.
4. Fake login forms can be injected to phish credentials.
5. Victims can be redirected to malicious or attacker-controlled sites.

## Remediation
1. Apply output encoding to the `cat` parameter wherever it is reflected.
2. Do not rely on keyword blacklisting (e.g., `confirm`); it can be bypassed via Unicode escapes, string concatenation, or alternate functions.
3. Validate and sanitize all parameters consistently, not just the ones tied to visible form fields.
4. Remove or disable undocumented/hidden parameters that aren't needed.
5. Set `HttpOnly`, `Secure`, and `SameSite` flags on session cookies.
