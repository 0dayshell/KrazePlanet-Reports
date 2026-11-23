## Stored XSS - Twitter

## Description
**Affected URL:** `https://ads.pwnedbox.site/`

A stored Cross-Site Scripting (XSS) vulnerability exists in the "New Network Report" feature of the application. User input submitted through the report name field is stored server-side without sanitization and rendered back into the page without output encoding whenever the report is displayed. Because the stored payload is executed on every page load for any user who views it, this differs from reflected XSS in that no crafted link needs to be sent to a specific victim.

This is a security issue because the injected script is persisted and executes automatically in the browser of every user who visits the affected page, running with the same privileges as the legitimate site's own code.

## Steps to Reproduce
1. Register an account and log in at `https://ads.pwnedbox.site/`.
2. Click the "New Network Report" button to open the report creation form.
3. In the report name field, enter the payload: `<img SrC=x oNErRor=confirm("navi")>`
4. Fill in any other required fields and submit the report.
5. Observe that a `confirm("navi")` dialog is displayed immediately upon submission, confirming the payload executed.
6. Visit `https://ads.pwnedbox.site/` again, and observe that the `confirm("navi")` dialog fires again automatically, confirming the payload is persistently stored and executes for any visitor.

## POC

Step 1: Submit a new network report with the payload `<img SrC=x oNErRor=confirm("navi")>` in the report name field.

<img width="959" height="527" alt="image" src="https://github.com/user-attachments/assets/0683a5cd-15b6-449b-9123-f2b0fff582f4" />

Step 2: Revisit `https://ads.pwnedbox.site/` and observe the stored payload executes automatically.

<img width="959" height="339" alt="image" src="https://github.com/user-attachments/assets/ad8d910d-a7ae-4ec7-829d-ce063639f337" />


**Result:** The unsanitized payload is stored server-side, and the `onerror` event handler fires automatically when the broken image fails to load, displaying a `confirm("navi")` dialog for every visitor, confirming stored XSS.

## Impact
1. Attacker-controlled script executes automatically for every user who views the report.
2. Session cookies/tokens can be stolen if not `HttpOnly`.
3. Victim accounts, including administrators, can be taken over via session hijacking.
4. Fake login forms can be injected to phish credentials from all visitors.
5. The application can be defaced or used to spread malware to all users.

## Remediation
1. Apply output encoding to the report name field wherever it is rendered.
2. Sanitize and validate input server-side before storing it, rejecting HTML/script content.
3. Strip or block dangerous HTML elements and event handler attributes (`onerror`, `<img>`, etc.).
4. Enforce a strict CSP to block inline event handlers and unauthorized script execution.
5. Set `HttpOnly`, `Secure`, and `SameSite` flags on session cookies.
