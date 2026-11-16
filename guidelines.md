## Reflected XSS in Search Functionality of Guidelines

## Description
**Affected URL:** `https://guidelines.pwnedbox.site/index.php`

A reflected Cross-Site Scripting (XSS) vulnerability exists in the search functionality of the application. The search feature takes user input through the `s` URL parameter and reflects it in the page response without proper input validation or output encoding. Because the input is rendered as raw HTML in the browser, an attacker can inject arbitrary JavaScript that executes in the context of the vulnerable domain.

This is a security issue because the injected script runs with the same privileges as the legitimate site's own code. The payload is delivered through a crafted link, so any user who opens that link can be targeted.

## Steps to Reproduce
1. Open the application in a browser: `https://guidelines.pwnedbox.site/index.php`
2. Locate the search field.
3. Enter a harmless test string such as `test` and submit the search.
4. Observe that the URL changes to `https://guidelines.pwnedbox.site/index.php?s=test` and the search term is reflected in the page.
5. Modify the `s` parameter to contain the following payload: <script>alert(1)</script>
6. Submit the request.
7. Observe that a JavaScript alert box with the value `1` is displayed, confirming the injected script was executed by the browser.

## POC

Step 1: Enter a harmless test string such as `test` and submit the search.

<img width="932" height="248" alt="image" src="https://github.com/user-attachments/assets/2b413728-2815-4e04-af56-47d18db0e96e" />


Observe that the URL changes to `https://guidelines.pwnedbox.site/index.php?s=test` and the search term is reflected in the page.

Step 2: Modify the `s` parameter to contain the following payload: <script>alert(1)</script>

<img width="930" height="221" alt="image" src="https://github.com/user-attachments/assets/1904dcde-df14-41bc-84ce-78f6379696f0" />


**Result:** The payload is returned unsanitized in the response and executed by the browser, displaying an alert popup.

## Impact
An attacker could craft a malicious link containing a JavaScript payload and deliver it to victims through phishing emails, messages, or other websites. When a victim opens the link, the script runs in their browser under the application's origin. Potential consequences include:

- **Session hijacking:** stealing session cookies or tokens not protected by the `HttpOnly` flag.
- **Account takeover:** performing actions on behalf of the victim using their authenticated session.
- **Phishing and credential theft:** injecting fake login forms or content into the trusted page.
- **Malicious redirects:** sending users to attacker-controlled sites.
- **Page defacement and data exposure:** modifying page content or reading sensitive data in the DOM.

Any user tricked into clicking the crafted link is affected, including administrators if they are targeted.

## Remediation
1. **Output encoding:** HTML-encode all user-controlled data (`<`, `>`, `"`, `'`, `&`) before rendering it, and use the templating engine's auto-escaping.
2. **Input validation:** Validate the `s` parameter against an allowlist of expected characters and length on the server side.
3. **Avoid unsafe DOM sinks:** If handled client-side, use `textContent` instead of `innerHTML` or `document.write`.
4. **Content Security Policy (CSP):** Disallow inline scripts (e.g., `script-src 'self'`) to reduce the impact of any injection.
5. **Cookie hardening:** Set `HttpOnly`, `Secure`, and `SameSite` attributes on session cookies.
6. **Security testing:** Include XSS checks in code review and automated testing, and use a trusted sanitization library wherever HTML input must be allowed.
