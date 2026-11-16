## Reflected XSS in Search Parameter

## Description
Affected URL: https://tutorialrepublic.pwnedbox.site/

A reflected Cross-Site Scripting (XSS) vulnerability exists in the search functionality of the application. The q GET parameter is taken from the URL and included in the page response without proper input validation or output encoding. Because the input is rendered as raw HTML in the browser, an attacker can inject arbitrary JavaScript that executes in the context of the vulnerable domain.

This is a security issue because the injected script runs with the same privileges as the legitimate site’s own code. The payload is delivered through a crafted link, so any user who opens that link can be targeted.

## Steps to Reproduce
1. Open the application in a browser: https://tutorialrepublic.pwnedbox.site/
2. Locate the search field.
3. Enter a harmless test string such as test and press Enter.
4. Observe that the URL changes to https://tutorialrepublic.pwnedbox.site/?q=test and the search term is reflected in the page.
5. Modify the q parameter to contain the following payload: <script>alert(1)</script>
6. Submit the request.
7. Observe that a JavaScript alert box with the value 1 is displayed, confirming the injected script was executed by the browser.

## POC (Proof of Concept)
Step 1: Enter a harmless test string such as test and press Enter.

<img width="893" height="368" alt="image" src="https://github.com/user-attachments/assets/50de9d68-957f-452c-afcc-801216e72f02" />

Observe that the URL changes to https://tutorialrepublic.pwnedbox.site/?q=test and the search term is reflected in the page.

Step 2: Modify the q parameter to contain the following payload: <script>alert(1)</script> and submit the request.

<img width="746" height="254" alt="image" src="https://github.com/user-attachments/assets/96272a18-7cc5-424c-921c-7f0d9e6a56fe" />

GET /?q=<script>alert(1)</script> HTTP/1.1  
Host: tutorialrepublic.pwnedbox.site

The payload is returned unsanitized in the response and executed by the browser, displaying an alert popup.


## Impact
An attacker could craft a malicious link containing a JavaScript payload and deliver it to victims through phishing emails, messages, or other websites. When a victim opens the link, the script runs in their browser under the application’s origin. Potential consequences include:

- Session hijacking: stealing session cookies or tokens not protected by the HttpOnly flag.
- Account takeover: performing actions on behalf of the victim using their authenticated session.
- Phishing and credential theft: injecting fake login forms or content into the trusted page.
- Malicious redirects: sending users to attacker-controlled sites.
- Page defacement and data exposure: modifying page content or reading sensitive data in the DOM.

Any user tricked into clicking the crafted link is affected, including administrators if they are targeted.

## Remediation
- Output encoding: HTML-encode all user-controlled data (<, >, ", ', &) before rendering it, and use the templating engine’s auto-escaping.
- Input validation: Validate the q parameter against an allowlist of expected characters and length on the server side.
- Avoid unsafe DOM sinks: If handled client-side, use textContent instead of innerHTML or document.write.
- Content Security Policy (CSP): Disallow inline scripts (e.g., script-src 'self') to reduce the impact of any injection.
- Cookie hardening: Set HttpOnly, Secure, and SameSite attributes on session cookies.
- Security testing: Include XSS checks in code review and automated testing, and use a trusted sanitization library wherever HTML input must be allowed.
