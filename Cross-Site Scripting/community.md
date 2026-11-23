## Stored XSS - Forum Discussions (Reddit Clone)

## Description
**Affected URL:** `https://community.pwnedbox.site/index.php`

A stored Cross-Site Scripting (XSS) vulnerability exists in the registration feature of the application. User input submitted through the "Organization / Institution" field is stored server-side without sanitization and rendered back unencoded on the user's Profile page. While several other fields in the registration form were also tested with similar payloads, only the "Organization / Institution" field reflects the stored value without encoding and executes it.

This is a security issue because the injected script is persisted in the account data and executes automatically whenever the Profile page is loaded, running with the same privileges as the legitimate site's own code.

## Steps to Reproduce
1. Open the application in a browser: `https://community.pwnedbox.site/index.php?view=register`
2. Fill in the registration form, entering the payload `navi6'"<ScRiPt>confirm(6)</ScRiPt>` in the "Organization / Institution" field (other fields can be filled with similar test payloads like `navi1'"<ScRiPt>confirm(1)</ScRiPt>`,`navi2'"<ScRiPt>confirm(2)</ScRiPt>` etc  to check for additional injection points).
3. Complete the remaining required fields (Email, Password, Captcha, etc.) and submit the registration form.
4. Log in with the newly created account.
5. Navigate to the "Profile" tab.
6. Observe that a `confirm(6)` dialog is displayed automatically, confirming the payload stored in the "Organization / Institution" field executed.

## POC

Step 1: Register with the payload `navi6'"<ScRiPt>confirm(6)</ScRiPt>` in the "Organization / Institution" field.

<img width="950" height="475" alt="image" src="https://github.com/user-attachments/assets/6301c013-031e-4ddb-83fb-50eeff53c220" />  

<img width="953" height="365" alt="image" src="https://github.com/user-attachments/assets/479ed72f-ef60-4727-a517-e3cad5e159a3" />

Step 2: Log in and navigate to the Profile tab.

<img width="956" height="482" alt="image" src="https://github.com/user-attachments/assets/8a872b33-5db2-4d3d-9b94-c958d997ab3a" />

<img width="959" height="536" alt="image" src="https://github.com/user-attachments/assets/4e397c07-0a7b-44af-b6f1-74da9207b567" />

**Result:** The "Organization / Institution" value is stored unsanitized and rendered unencoded on the Profile page, executing the injected `<ScRiPt>` tag and displaying a `confirm(6)` dialog, confirming stored XSS.

## Impact
1. Attacker-controlled script executes automatically whenever the Profile page is viewed.
2. Session cookies/tokens can be stolen if not `HttpOnly`.
3. The account owner (or anyone with access to view the profile, e.g. an admin) can have their session hijacked.
4. Fake login forms can be injected to phish credentials from anyone viewing the profile.
5. The application can be defaced or used to spread malware via the profile page.

## Remediation
1. Apply output encoding to the "Organization / Institution" field wherever it is rendered.
2. Sanitize and validate input server-side at registration, rejecting HTML/script content.
3. Restrict the field to a safe allowlist of characters (e.g., letters, numbers, spaces, basic punctuation).
4. Enforce a strict CSP to block inline script execution.
5. Set `HttpOnly`, `Secure`, and `SameSite` flags on session cookies.
