## Reflected XSS - Mixed Security Parameters

## Description
**Affected URL:** `https://settings.pwnedbox.site/`

A reflected Cross-Site Scripting (XSS) vulnerability exists in a hidden `academy` parameter of the application. The visible `fname` and `lname` parameters are filtered/input-validated, but parameter enumeration revealed a large number of undocumented parameters on the same endpoint (including `api`, `callback`, `email`, `id`, `q`, `search`, `token`, `url`, `username`, and others). The `academy` parameter reflects user input into the page response without any validation, and a JavaScript Unicode escape sequence bypasses the keyword-based filter blocking `confirm`.

This is a security issue because an undocumented parameter bypasses the input validation applied to the visible form fields, and the injected script runs with the same privileges as the legitimate site's own code.

## Steps to Reproduce
1. Open the application in a browser: `https://settings.pwnedbox.site/`
2. Enter the payload `navi'"<>` into the `fname` and `lname` fields and submit. Observe that the characters are filtered/sanitized in the response.
3. Perform parameter enumeration and discover a large number of undocumented parameters on the same endpoint, including `academy`.
4. Submit the test payload `navi'"<>` via the `academy` parameter: `https://settings.pwnedbox.site/?academy=navi'"<>` and observe that it is reflected on the page without filtering.
5. Replace `confirm` with its JavaScript Unicode escape sequence equivalent to bypass the keyword filter: `navi35'"<><ScRiPt>\u0063onfirm("navi35")</ScRiPt>`.
6. Submit the request and observe that a `confirm("navi35")` dialog is displayed, confirming successful reflected XSS execution via the Unicode escape filter bypass.

## POC

Step 1: Submit `navi'"<>` via `fname`/`lname` to confirm those fields are filtered.

<img width="950" height="458" alt="image" src="https://github.com/user-attachments/assets/b5022742-bdf8-442b-83a5-de3af953efae" />

Step 2: Submit `navi'"<>` via the discovered `academy` parameter to confirm it is reflected unsanitized.

<img width="959" height="230" alt="image" src="https://github.com/user-attachments/assets/eadaff84-5913-4623-b238-4b46667f1c73" />

Step 3: Submit the payload via the `academy` parameter: navi'""<><ScRiPt>\u0063onfirm("navi35")</ScRiPt>

<img width="957" height="371" alt="image" src="https://github.com/user-attachments/assets/8246eb57-c154-4485-b9c2-8416f5589873" />

**Result:** The `confirm` keyword filter does not account for the JavaScript Unicode escape sequence `\u0063onfirm`, and the payload executes, displaying a `confirm("navi35")` dialog, confirming reflected XSS via the `academy` parameter.

## Impact
1. Attacker-controlled script executes in the victim's browser session.
2. Session cookies/tokens can be stolen if not `HttpOnly`.
3. Victim accounts can be taken over via session hijacking.
4. Fake login forms can be injected to phish credentials.
5. Victims can be redirected to malicious or attacker-controlled sites.

## Remediation
1. Apply output encoding to the `academy` parameter wherever it is reflected.
2. Do not rely on keyword blacklisting (e.g., `confirm`); it can be bypassed via Unicode escapes, string concatenation, or alternate functions.
3. Validate and sanitize all parameters consistently, not just the ones tied to visible form fields.
4. Remove or disable undocumented/hidden parameters that aren't needed; a large attack surface of unused parameters increases risk.
5. Set `HttpOnly`, `Secure`, and `SameSite` flags on session cookies.
