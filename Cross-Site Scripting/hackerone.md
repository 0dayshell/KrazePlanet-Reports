## Stored XSS - Bug Bounty Platform (HackerOne Clone)

## Description
**Affected URL:** `https://hackerone.pwnedbox.site/index.php`

A stored Cross-Site Scripting (XSS) vulnerability exists in the "Submit Report" feature of the application. User input submitted through the report Title, Description, and Impact fields is stored server-side without sanitization and rendered back into the page without output encoding whenever the report is opened/viewed. The application appears to filter the literal lowercase `<script>` tag, but this blacklist-based filter is case-sensitive and does not account for mixed-case variations, allowing the filter to be bypassed.

This is a security issue because the injected script is persisted in the report data and executes automatically for any user who opens the report, running with the same privileges as the legitimate site's own code.

## Steps to Reproduce
1. Register an account and log in at `https://hackerone.pwnedbox.site/index.php`.
2. Click the "Submit Report" button to open the report creation form.
3. Fill in the form with the following values:
   - Title: `navi1'"<><ScRipT>confirm("navi")</ScRipT>`
   - Program: `Matomo`
   - Severity: `Medium`
   - Description: `navi1'"<><ScRipT>confirm("navi")</ScRipT>`
   - Impact: `navi1'"<><ScRipT>confirm("navi")</ScRipT>`
4. Submit the report and observe that it is listed among other reports.
5. Open/click on the submitted report and observe that a `confirm("navi")` dialog is displayed, confirming the payload executed.
6. Have another user/session open the same report and observe that the `confirm("navi")` dialog fires again, confirming the payload is persistently stored and executes for any viewer.

## POC

Step 1: Submit a report with the payload `navi1'"<><ScRipT>confirm("navi")</ScRipT>` in the Title, Description, and Impact fields.

<img width="937" height="531" alt="image" src="https://github.com/user-attachments/assets/7b01ad19-9794-4bef-a5a8-76b6f1f842da" />

Step 2: Open the submitted report from the reports list and observe the stored payload executes automatically.

<img width="953" height="430" alt="image" src="https://github.com/user-attachments/assets/65e08a47-1d88-45a1-a8e2-136148b29dbc" />


<img width="957" height="430" alt="image" src="https://github.com/user-attachments/assets/b94c13ca-0a62-4a70-951d-fa348c8a93db" />


**Result:** The case-sensitive `script` filter does not block the mixed-case `ScRipT` tag; the unsanitized payload is stored server-side and executes whenever the report is viewed, displaying a `confirm("navi")` dialog, confirming stored XSS.

## Impact
1. Attacker-controlled script executes automatically for every user who opens the report.
2. Session cookies/tokens can be stolen if not `HttpOnly`.
3. Any user's account, including administrators/triagers, can be taken over via session hijacking.
4. Fake login forms can be injected to phish credentials from report viewers.
5. The application can be defaced or used to spread malware to all users who view reports.

## Remediation
1. Do not rely on case-sensitive keyword blacklisting (`<script>`); use proper output encoding instead.
2. Apply output encoding to the Title, Description, and Impact fields wherever they are rendered.
3. Sanitize and validate input server-side before storing it, rejecting HTML/script content.
4. Enforce a strict CSP to block inline script execution regardless of filter bypasses.
5. Set `HttpOnly`, `Secure`, and `SameSite` flags on session cookies.
