## Reflected XSS - Function Name Filter

## Description
**Affected URL:** `https://profile.pwnedbox.site/`

A reflected Cross-Site Scripting (XSS) vulnerability exists in the GET Parameter Reflection feature of the application. The `fname` parameter filters special characters such as `'`, `"`, `<`, `>`, but the `lname` parameter does not sanitize these characters, allowing HTML/JavaScript injection. The application also attempts to block the keyword `alert`, but this blacklist-based filter does not account for other JavaScript functions that trigger dialog boxes, such as `prompt()`, allowing the filter to be bypassed.

This is a security issue because the injected script runs with the same privileges as the legitimate site's own code, and inconsistent validation between parameters along with an incomplete keyword blacklist leaves the application exploitable.

## Steps to Reproduce
1. Open the application in a browser: `https://profile.pwnedbox.site/`
2. Enter the payload `navi'"<>` into the First Name field and submit. Observe that the special characters are filtered/sanitized in the response.
3. Enter the payload `navi"><script>alert(1)</script>` into the Last Name field and submit. Observe that the characters pass through, but the word `alert` is stripped/blocked, preventing execution.
4. Replace `alert(1)` with `prompt(1)` in the Last Name field: `navi"><script>prompt(1)</script>`. Submit the request with the URL `https://profile.pwnedbox.site/?fname=navi%27%22%3C%3E&lname=navi%22%3E%3Cscript%3Eprompt%281%29%3C%2Fscript%3E`.
5. Observe that a JavaScript prompt dialog is displayed, confirming successful reflected XSS execution via the `lname` parameter using the `alert` filter bypass.

## POC

Step 1: Submit `navi'"<>` in the both fields and confirm the characters are filtered.

<img width="713" height="478" alt="image" src="https://github.com/user-attachments/assets/1de316a9-f9c1-464b-b555-a1593cb08e21" />


Step 2: Submit `navi"><script>alert(1)</script>` in the Last Name field and confirm `alert` is blocked while other characters pass through.

<img width="821" height="175" alt="image" src="https://github.com/user-attachments/assets/e88f15e5-a3a0-44d6-8049-6211755b684d" />

Step 3: Submit `navi"><script>prompt(1)</script>` in the Last Name field using the URL:

<img width="717" height="251" alt="image" src="https://github.com/user-attachments/assets/0aa5c8db-a03e-41b8-a320-bddffa7b6a4c" />

**Result:** The `alert` keyword filter does not block `prompt`, and the injected script executes, displaying a JavaScript prompt dialog, confirming reflected XSS via the `lname` parameter.

## Impact
1. Attacker-controlled script executes in the victim's browser session.
2. Session cookies/tokens can be stolen if not `HttpOnly`.
3. Victim accounts can be taken over via session hijacking.
4. Fake login forms can be injected to phish credentials.
5. Victims can be redirected to malicious or attacker-controlled sites.

## Remediation
1. Apply consistent output encoding to all parameters, including `lname`.
2. Replace keyword blacklisting (e.g., blocking only `alert`) with proper HTML/attribute encoding.
3. Validate all user input against an allowlist of expected characters.
4. Enforce a strict CSP to block inline script execution regardless of function used.
5. Set `HttpOnly`, `Secure`, and `SameSite` flags on session cookies.
