## Stored XSS - Video Streaming Service (Netflix Clone)

## Description
**Affected URL:** `https://netflix.pwnedbox.site/index.php`

A stored Cross-Site Scripting (XSS) vulnerability exists in the signup feature of the application. User input submitted through the "Full Name" and "Username" fields during registration is stored server-side without sanitization and rendered back into the index page without output encoding. The application appears to filter the literal lowercase `<script>` tag, but this blacklist-based filter is case-sensitive and does not account for mixed-case variations, allowing the filter to be bypassed.

This is a security issue because the injected script is persisted in the user's account data and executes automatically every time the index page is loaded, for any user who views it, running with the same privileges as the legitimate site's own code.

## Steps to Reproduce
1. Open the application in a browser: `https://netflix.pwnedbox.site/index.php`
2. Click the "Sign In" button, then switch to the "Sign Up" form.
3. In the Full Name and Username fields, enter the payload: `navi'"<><ScRipT>confirm("navi")</ScRipT>`, using mixed case to bypass the case-sensitive `script` filter.
4. Fill in the remaining required fields (email, password) and submit the signup form.
5. Observe that a `confirm("navi")` dialog is displayed immediately after signup, confirming the payload executed.
6. Refresh the index page and observe that the `confirm("navi")` dialog fires again automatically on every reload, confirming the payload is persistently stored and executes on every page load.

## POC

Step 1: Sign up with the payload `navi'"<><ScRipT>confirm("navi")</ScRipT>` in the Full Name and Username fields.

<img width="959" height="508" alt="image" src="https://github.com/user-attachments/assets/703d0373-1465-4658-a45b-5e05269c2cb3" />

Step 2: Refresh the index page and observe the stored payload executes automatically.

<img width="959" height="452" alt="image" src="https://github.com/user-attachments/assets/a61c4c84-4610-44bd-a957-3d31624ed6ad" />

**Result:** The case-sensitive `script` filter does not block the mixed-case `ScRipT` tag; the unsanitized payload is stored server-side and executes on every page load, displaying a `confirm("navi")` dialog, confirming stored XSS.

## Impact
1. Attacker-controlled script executes automatically for every user who loads the index page.
2. Session cookies/tokens can be stolen if not `HttpOnly`.
3. Any user's account, including administrators, can be taken over via session hijacking.
4. Fake login forms can be injected to phish credentials from all visitors.
5. The application can be defaced or used to spread malware to all users.

## Remediation
1. Do not rely on case-sensitive keyword blacklisting (`<script>`); use proper output encoding instead.
2. Apply output encoding to the Full Name and Username fields wherever they are rendered.
3. Sanitize and validate input server-side before storing it, rejecting HTML/script content.
4. Enforce a strict CSP to block inline script execution regardless of filter bypasses.
5. Set `HttpOnly`, `Secure`, and `SameSite` flags on session cookies.
