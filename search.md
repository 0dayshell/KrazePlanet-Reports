## Reflected XSS - Global Site Search Portal

## Description
**Affected URL:** `https://search.pwnedbox.site/`

A reflected Cross-Site Scripting (XSS) vulnerability exists in the search functionality of the application. User input submitted through the `fname` and `lname` URL parameters is reflected back into the page inside the `value` attribute of their respective `<input>` elements without proper input validation or output encoding. Because the input is inserted directly into an HTML attribute, an attacker can break out of the attribute context and inject arbitrary HTML/JavaScript that executes in the context of the vulnerable domain.

This is a security issue because the injected script runs with the same privileges as the legitimate site's own code. The payload is delivered through a crafted link, so any user who opens that link can be targeted.

## Steps to Reproduce
1. Open the application in a browser: `https://search.pwnedbox.site/`
2. Locate the "First Name" and "Last Name" input fields.
3. Enter a special character test payload such as `navi'"<>` into both fields and submit the form.
4. Observe that the URL changes to `https://search.pwnedbox.site/?fname=navi%27%22%3C%3E&lname=navi%27%22%3C%3E` and the payload is reflected back on the page, confirming the input is not sanitized.
5. Modify the payload to `navi<script>alert(1)</script>` and submit. The URL becomes `https://search.pwnedbox.site/?fname=navi%3Cscript%3Ealert%281%29%3C%2Fscript%3E&lname=navi%3Cscript%3Ealert%281%29%3C%2Fscript%3E`.
6. Inspect the page source and observe the payload is reflected unencoded inside the `value` attribute of the input field:
   ```html
   <input class="form-control search-box" type="text" id="fname" placeholder="Enter first name" name="fname" value="navi<script>alert(1)</script>">
   ```
   The script tag does not execute here because it is placed inside an attribute value rather than the HTML body.
7. Modify the payload to break out of the attribute context: `navi"><script>alert(1)</script>`
8. Submit the request and observe that a JavaScript alert box with the value `1` is displayed, confirming successful reflected XSS execution.

## POC

Step 1: Enter the payload `navi'"<>` into the First Name and Last Name fields and submit.

<img width="955" height="485" alt="image" src="https://github.com/user-attachments/assets/ae08b4ac-85bb-43ea-883d-e650ba1d4bfe" />


Observe that the URL changes to `https://search.pwnedbox.site/?fname=navi%27%22%3C%3E&lname=navi%27%22%3C%3E` and the payload is reflected on the page.

Step 2: Modify the payload to `navi<script>alert(1)</script>` and submit.

<img width="956" height="464" alt="image" src="https://github.com/user-attachments/assets/fc271b51-858f-4bd0-a10f-36ac70651069" />


Inspect the page source to confirm the payload lands inside the `value` attribute without encoding:

```html
<input class="form-control search-box" type="text" id="fname" placeholder="Enter first name" name="fname" value="navi<script>alert(1)</script>">
```

<img width="820" height="143" alt="image" src="https://github.com/user-attachments/assets/4269c9ae-5b22-4a96-bc97-17b5043ce091" />

The <script> tag is reflected literally as text inside the value attribute, so the browser parses it as part of the attribute string rather than as HTML markup, and it never executes.

Step 3: Modify the payload to `navi"><script>alert(1)</script>` to break out of the `value` attribute and inject a new executable script tag.

<img width="956" height="472" alt="image" src="https://github.com/user-attachments/assets/4d628c63-b673-4a05-8f1c-c0e7673f9cfe" />

<img width="959" height="251" alt="image" src="https://github.com/user-attachments/assets/00d15a33-3ad2-4cbf-8ef5-b4f70405eebd" />


**Result:** The payload breaks out of the `value` attribute and the injected `<script>` tag executes, displaying an `alert(1)` popup, confirming reflected XSS.

## Impact
An attacker could craft a malicious link containing a JavaScript payload and deliver it to victims through phishing emails, messages, or other websites. When a victim opens the link, the script runs in their browser under the application's origin. Potential consequences include:

- **Session hijacking:** stealing session cookies or tokens not protected by the `HttpOnly` flag.
- **Account takeover:** performing actions on behalf of the victim using their authenticated session.
- **Phishing and credential theft:** injecting fake login forms or content into the trusted page.
- **Malicious redirects:** sending users to attacker-controlled sites.
- **Page defacement and data exposure:** modifying page content or reading sensitive data in the DOM.

Any user tricked into clicking the crafted link is affected, including administrators if they are targeted.

## Remediation
1. **Output encoding:** HTML-attribute-encode all user-controlled data (`<`, `>`, `"`, `'`, `&`) before inserting it into attribute values, and use the templating engine's auto-escaping instead of manual string concatenation.
2. **Input validation:** Validate the `fname` and `lname` parameters against an allowlist of expected characters (e.g., letters, spaces, hyphens) and reasonable length limits on the server side.
3. **Avoid unsafe DOM sinks:** If handled client-side, use `textContent`/`setAttribute` with encoded values instead of directly concatenating user input into HTML strings.
4. **Content Security Policy (CSP):** Disallow inline scripts (e.g., `script-src 'self'`) to reduce the impact of any injection that bypasses encoding.
5. **Cookie hardening:** Set `HttpOnly`, `Secure`, and `SameSite` attributes on session cookies.
6. **Security testing:** Include XSS checks (including attribute-breakout payloads) in code review and automated testing, and use a trusted sanitization/encoding library wherever user input is placed into HTML.
