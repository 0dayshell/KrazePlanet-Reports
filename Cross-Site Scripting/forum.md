## Stored XSS - Acronis

## Description
**Affected URL:** `https://forum.pwnedbox.site/`

A stored Cross-Site Scripting (XSS) vulnerability exists in the user profile feature of the application. User input submitted through the "Display Name", "About Me", and "Profile Signature" fields under Public Information is stored server-side without sanitization and rendered back unencoded on the profile page. Each field independently reflects and executes the stored payload.

This is a security issue because the injected payloads are persisted in the account data and execute automatically whenever the profile page is loaded, running with the same privileges as the legitimate site's own code.

## Steps to Reproduce
1. Log in to `https://forum.pwnedbox.site/` using test credentials (Alice, alice@devask.io and password alice123).
2. Navigate to the Profile page.
3. Enter the payload `navi'"<><iMg srC=x onerrOr=confirm(1)>` into the Display Name, `navi'"<><iMg srC=x onerrOr=confirm(2)>` into the About Me, and `navi'"<><iMg srC=x onerrOr=confirm(3)>` into Profile Signature fields.
4. Click "Save Profile".
5. Observe that the `3` is displayed in sequence, confirming Profile Signature field is vulnerable to store and execute the injected payload independently.

## POC

Step 1: Add the payload `navi'"<><iMg srC=x onerrOr=confirm(1)>` into the Display Name, `navi'"<><iMg srC=x onerrOr=confirm(2)>` into the About Me, and `navi'"<><iMg srC=x onerrOr=confirm(3)>` into Profile Signature fields.

<img width="957" height="496" alt="image" src="https://github.com/user-attachments/assets/c32c07de-dccb-43d3-8970-3e364af523e7" />

Step 2: Submit the update info 

<img width="956" height="535" alt="image" src="https://github.com/user-attachments/assets/a2492e43-ba75-4afd-a826-b17c2f51a352" />

**Result:** the `3` fire in sequence upon loading the profile page, confirming stored XSS in Profile Signature field.

## Impact
1. Attacker-controlled script executes automatically whenever the profile page is viewed.
2. Session cookies/tokens can be stolen if not `HttpOnly`.
3. The account owner, or any other user/admin who views the profile, can have their session hijacked.
4. Fake login forms can be injected to phish credentials from anyone viewing the profile.
5. The application can be defaced or used to spread malware via the profile page.

## Remediation
1. Apply output encoding to the Display Name, About Me, and Profile Signature fields wherever they are rendered.
2. Sanitize and validate input server-side before storing it, rejecting HTML/script content and event handler attributes.
3. Restrict these fields to a safe allowlist of characters where possible.
4. Enforce a strict CSP to block inline event handlers like `onerror`.
5. Set `HttpOnly`, `Secure`, and `SameSite` flags on session cookies.
