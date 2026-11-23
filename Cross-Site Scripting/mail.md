## Reflected XSS - Encoding Bypass Attempts

## Description
**Affected URL:** `https://mail.pwnedbox.site/`

A reflected Cross-Site Scripting (XSS) vulnerability exists in a hidden `cat` parameter of the application. The visible `fname` and `lname` parameters are filtered/input-validated, but parameter enumeration revealed multiple undocumented parameters on the same endpoint, including `cat`, `id`, `number`, `page`, and `page.id`. The `cat` parameter reflects user input into the page response without any validation, and while the application blocks the literal keyword `confirm`, this blacklist-based filter does not account for JavaScript Unicode escape sequences, allowing the filter to be bypassed.

This is a security issue because an undocumented parameter bypasses the input validation applied to the visible form fields, and the injected script runs with the same privileges as the legitimate site's own code.

## Steps to Reproduce
1. Open the application in a browser: `https://mail.pwnedbox.site/`
2. Enter the payload `navi'"<>` into the `fname` and `lname` fields and submit. Observe that the characters are filtered/sanitized in the response.
3. Perform parameter enumeration and discover additional undocumented parameters on the same endpoint: `cat`, `id`, `number`, `page`, `page.id`.
4. Submit the test payload `navi'"<>` via the `cat` parameter: `https://mail.pwnedbox.site/?cat=navi'"<>` and observe that it is reflected on the page without filtering.
5. Submit the payload `navi'"<><ScRiPt>confirm(1)</ScRiPt>` via the `cat` parameter and observe that the keyword `confirm` is stripped/blocked, preventing execution.
6. Replace `confirm` with its JavaScript Unicode escape sequence equivalent: `navi'"<><ScRiPt>\u0063onfirm(1)</ScRiPt>`.
7. Submit the request and observe that a `confirm(1)` dialog is displayed, confirming successful reflected XSS execution via the Unicode escape filter bypass.

## POC

Step 1: Submit `navi'"<>` via `fname`/`lname` to confirm those fields are filtered.

<img width="957" height="463" alt="image" src="https://github.com/user-attachments/assets/91e1bb30-e9f0-4e51-9076-a62f6e64e22e" />

Step 2: Submit `navi'"<>` via the discovered `cat` parameter to confirm it is reflected unsanitized.



Step 3: Submit `navi'"<><ScRiPt>confirm(1)</ScRiPt>` via `cat` and confirm the keyword `confirm` is blocked.

**[Insert screenshot here]**

Step 4: Submit the payload via the `cat` parameter:



<img width="959" height="212" alt="image" src="https://github.com/user-attachments/assets/1e7755d3-58fd-4e36-978e-ab0b90acdc13" />


<img width="959" height="331" alt="image" src="https://github.com/user-attachments/assets/b6afd9c6-1914-41a2-9a0a-6cd746a16a25" />
