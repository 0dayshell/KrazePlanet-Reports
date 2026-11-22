## Reflected XSS - Multi-Parameter Filter Evasion

## Description
**Affected URL:** `https://checkout.pwnedbox.site/`

A reflected Cross-Site Scripting (XSS) vulnerability exists in a hidden `cat` parameter of the application. The visible `fname` and `lname` parameters are filtered/input-validated, but parameter enumeration revealed multiple undocumented parameters on the same endpoint, including `cat`, `id`, `number`, `page`, and `page.id`. The `cat` parameter reflects user input into the page response without any validation or output encoding, and the application's keyword-based filter is case-sensitive, allowing a mixed-case `<sCrIpt>` tag to bypass it.

This is a security issue because an undocumented parameter bypasses the input validation applied to the visible form fields, and the injected script runs with the same privileges as the legitimate site's own code.

## Steps to Reproduce
1. Open the application in a browser: `https://checkout.pwnedbox.site/`
2. Enter the payload `navi'"<>` into the `fname` and `lname` fields and submit. Observe that the characters are filtered/sanitized in the response.
3. Perform parameter enumeration and discover additional undocumented parameters on the same endpoint: `cat`, `id`, `number`, `page`, `page.id`.
4. Isolate the `cat` parameter by requesting `https://checkout.pwnedbox.site/?cat=` and submit the test payload `navi'"<>`. Observe that it is reflected on the page without filtering.
5. Submit the payload `navi'"<><sCrIpt>confirm(1)</sCrIpt>` via the `cat` parameter, using mixed case to bypass the case-sensitive `script` filter.
6. Submit the request and observe that a `confirm(1)` dialog is displayed, confirming successful reflected XSS execution via the hidden `cat` parameter.

## POC

Step 1: Submit `navi'"<>` via `fname`/`lname` to confirm those fields are filtered.

<img width="571" height="466" alt="image" src="https://github.com/user-attachments/assets/2fa452ad-22e1-4d17-9db8-70232f3f90a5" />

Step 2: Enumerate parameters and isolate `cat`, then submit `navi'"<>` via `https://checkout.pwnedbox.site/?cat=` to confirm it is reflected unsanitized.

<img width="959" height="496" alt="image" src="https://github.com/user-attachments/assets/f590e160-fa39-4643-9f5c-738566252b1b" />


Step 3: Submit the payload via the `cat` parameter:

<img width="953" height="470" alt="image" src="https://github.com/user-attachments/assets/efd97d9f-5e8f-49c0-99f3-b0c0d4c4d6c5" />
