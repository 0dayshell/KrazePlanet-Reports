## Stored XSS - Support Center (DigitalOcean Clone)

## Description
**Affected URL:** `https://digitalocean.pwnedbox.site/index.php`

A stored Cross-Site Scripting (XSS) vulnerability exists in the "Create Ticket" feature of the application. User input submitted through the ticket Subject and Description fields is stored server-side without sanitization and rendered back into the "View Tickets" page without output encoding. The application appears to filter the literal lowercase `<script>` tag, but this blacklist-based filter is case-sensitive and does not account for mixed-case variations, allowing the filter to be bypassed.

This is a security issue because the injected script is persisted in the ticket data and executes automatically for any user who visits the View Tickets page, running with the same privileges as the legitimate site's own code.

## Steps to Reproduce
1. Register an account and log in at `https://digitalocean.pwnedbox.site/index.php`.
2. Click the "Create Ticket" button to open the ticket creation form.
3. Fill in the form with the following values:
   - Subject: `navi1'"<><ScRipT>confirm(1)</ScRipT>`
   - Priority: `Critical`
   - Description: `navi1'"<><ScRipT>confirm(1)</ScRipT>`
4. Submit the ticket.
5. Visit the "View Tickets" page at `https://digitalocean.pwnedbox.site/index.php?view=tickets` and observe that a `confirm(1)` dialog is displayed, confirming the payload executed.
6. Have another user/session visit the same page and observe that the `confirm(1)` dialog fires again, confirming the payload is persistently stored and executes for any visitor.

## POC

Step 1: Submit a ticket with the payload `navi1'"<><ScRipT>confirm(1)</ScRipT>` in the Subject and Description fields.

<img width="959" height="515" alt="image" src="https://github.com/user-attachments/assets/c7553124-9453-4f8f-9014-e11e5c454828" />

Step 2: Visit `https://digitalocean.pwnedbox.site/index.php?view=tickets` and observe the stored payload executes automatically.

<img width="959" height="481" alt="image" src="https://github.com/user-attachments/assets/010b3782-bc3a-4bf4-9162-75938a162472" />

**Result:** The case-sensitive `script` filter does not block the mixed-case `ScRipT` tag; the unsanitized payload is stored server-side and executes whenever the View Tickets page is loaded, displaying a `confirm(1)` dialog, confirming stored XSS.

## Impact
1. Attacker-controlled script executes automatically for every user who visits the View Tickets page.
2. Session cookies/tokens can be stolen if not `HttpOnly`.
3. Any user's account, including administrators/support staff, can be taken over via session hijacking.
4. Fake login forms can be injected to phish credentials from ticket viewers.
5. The application can be defaced or used to spread malware to all users who view tickets.

## Remediation
1. Do not rely on case-sensitive keyword blacklisting (`<script>`); use proper output encoding instead.
2. Apply output encoding to the Subject and Description fields wherever they are rendered.
3. Sanitize and validate input server-side before storing it, rejecting HTML/script content.
4. Enforce a strict CSP to block inline script execution regardless of filter bypasses.
5. Set `HttpOnly`, `Secure`, and `SameSite` flags on session cookies.
