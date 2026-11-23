## Reflected XSS - Category Filter

## Description
**Affected URL:** `https://shop.pwnedbox.site/`

A reflected Cross-Site Scripting (XSS) vulnerability exists in the category filtering feature of the application. Clicking a category (e.g. "Physics") sets the `category` URL parameter, and this value is reflected directly into the page response without any input validation or output encoding, allowing a crafted script payload to execute directly in the browser.

This is a security issue because the injected script runs with the same privileges as the legitimate site's own code. The payload is delivered through a crafted link, so any user who opens that link can be targeted.

## Steps to Reproduce
1. Open the application in a browser: `https://shop.pwnedbox.site/`
2. Click on a category, such as "Physics", which sets the URL to `https://shop.pwnedbox.site/?category=Physics`.
3. Modify the `category` parameter value to `navi'"<>` and observe that the payload is reflected on the page, confirming no input sanitization.
4. Modify the `category` parameter to the payload `navi'"<><ScRiPt>\u0063onfirm("navi")</ScRiPt>`.
5. Submit the request and observe that a `confirm("navi")` dialog is displayed, confirming successful reflected XSS execution.

## POC

Step 1: Modify the `category` parameter to `navi'"<>` to confirm the input is reflected unsanitized.

<img width="959" height="413" alt="image" src="https://github.com/user-attachments/assets/57c467e4-8352-4249-85d1-b3d34945c9f8" />

Step 2: Modify the `category` parameter to `navi'"<><ScRiPt>\u0063onfirm("navi")</ScRiPt>`:

<img width="959" height="256" alt="image" src="https://github.com/user-attachments/assets/f7333ab3-8c6f-4d5b-973c-bcbda7c0861a" />

**Result:** The payload is returned unsanitized in the response and executed by the browser, displaying a `confirm("navi")` dialog, confirming reflected XSS via the `category` parameter.

## Impact
1. Attacker-controlled script executes in the victim's browser session.
2. Session cookies/tokens can be stolen if not `HttpOnly`.
3. Victim accounts can be taken over via session hijacking.
4. Fake login forms can be injected to phish credentials.
5. Victims can be redirected to malicious or attacker-controlled sites.

## Remediation
1. Apply output encoding to the `category` parameter before rendering it in responses.
2. Validate the `category` value against an allowlist of permitted category names/IDs.
3. Avoid inserting unsanitized input into the DOM via unsafe sinks.
4. Enforce a strict CSP to block inline script execution.
5. Set `HttpOnly`, `Secure`, and `SameSite` flags on session cookies.
