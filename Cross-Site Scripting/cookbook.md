## Reflected XSS - Multi-Parameter Script Filter Evasion

## Title
REFLECTED XSS IN SEARCH PARAMETER

## Description
**Affected URL:** `https://feedback.pwnedbox.site/`

A reflected Cross-Site Scripting (XSS) vulnerability exists in the GET Parameter Reflection feature of the application. User input submitted through the `fname` and `lname` URL parameters is reflected back into the page inside the `value` attribute of their respective `<input>` elements. The application attempts to filter the input by stripping the literal string `script`, but the filter is case-sensitive and only strips the lowercase form, so it does not account for mixed-case variations. By using `<Script>` instead of `<script>`, the filter is bypassed, allowing the tag to break out of the attribute context and execute as HTML.

This is a security issue because the injected script runs with the same privileges as the legitimate site's own code, and a case-based filter bypass shows the sanitization is insufficient. The payload is delivered through a crafted link, so any user who opens that link can be targeted.

## Steps to Reproduce
1. Open the application in a browser: `https://feedback.pwnedbox.site/`
2. Locate the "First Name" and "Last Name" input fields.
3. Enter the payload `navi'"<>` into both fields and submit. The URL becomes `https://feedback.pwnedbox.site/?fname=navi%27"<>&lname=navi%27"<>`, and the payload is reflected on the page, confirming no input sanitization.
4. Modify the payload to `navi<script>alert(1)</script>` in both fields and submit. The URL becomes `https://feedback.pwnedbox.site/?fname=navi<script>alert(1)<%2Fscript>&lname=navi<script>alert(1)<%2Fscript>`.
5. Observe that the page displays `navi<>alert(1)</>` instead of an alert popup, and inspecting the source confirms the literal string `script` has been stripped from the input.
6. This shows the filter is case-sensitive and only removes the lowercase `script` string, so a mixed-case tag is not filtered.
7. In both the First Name and Last Name fields, submit the payload `navi"><Script>alert(1)</Script>` to break out of the attribute and bypass the case-sensitive filter.
8. Observe that JavaScript alert boxes with the value `1` are displayed, confirming successful reflected XSS execution in both fields.

## POC

Step 1: Enter the payload `navi'"<>` into both fields and submit to confirm the input is reflected unsanitized.

<img width="956" height="475" alt="image" src="https://github.com/user-attachments/assets/263d9887-f82e-4c98-ae1a-f1bfb69db1ae" />


Step 2: Modify the payload to `navi<script>alert(1)</script>` and submit. The rendered page shows `navi<>alert(1)</>` instead of the full tag.

<img width="956" height="470" alt="image" src="https://github.com/user-attachments/assets/c4b9ab25-8c8e-4c45-8725-bcb500dcb4ff" />


Inspecting the page source confirms the literal string `script` was stripped from the input, while the rest of the payload (`<`, `>`, `alert(1)`) passed through unfiltered:

<img width="777" height="158" alt="image" src="https://github.com/user-attachments/assets/4ace308c-1bd1-4803-b70b-f16950f2932f" />

Step 3: In both the First Name and Last Name fields, submit the payload `navi"><Script>alert(1)</Script>`, using mixed case to bypass the case-sensitive `script` filter.

<img width="958" height="489" alt="image" src="https://github.com/user-attachments/assets/2c537fb9-842f-43ed-9e50-0354e63be27d" />

**Result:** The case-sensitive filter does not strip `Script`, and the payload breaks out of the `value` attribute in both fields, each executing and displaying an `alert(1)` popup, confirming reflected XSS.

## Impact
An attacker could craft a malicious link containing a JavaScript payload and deliver it to victims through phishing emails, messages, or other websites. When a victim opens the link, the script runs in their browser under the application's origin. Potential consequences include:

- **Session hijacking:** stealing session cookies or tokens not protected by the `HttpOnly` flag.
- **Account takeover:** performing actions on behalf of the victim using their authenticated session.
- **Phishing and credential theft:** injecting fake login forms or content into the trusted page.
- **Malicious redirects:** sending users to attacker-controlled sites.
- **Page defacement and data exposure:** modifying page content or reading sensitive data in the DOM.

Any user tricked into clicking the crafted link is affected, including administrators if they are targeted.

## Remediation
1. **Do not rely on blacklisting:** Stripping the literal string `script` is insufficient and easily bypassed (case variation, nested tags, event handlers, other executable contexts). Replace this approach with proper output encoding.
2. **Output encoding:** HTML-attribute-encode all user-controlled data (`<`, `>`, `"`, `'`, `&`) before inserting it into attribute values, using the templating engine's auto-escaping rather than manual filtering.
3. **Input validation:** Validate the `fname` and `lname` parameters against an allowlist of expected characters (e.g., letters, spaces, hyphens) on the server side.
4. **Avoid unsafe DOM sinks:** If handled client-side, use `textContent`/`setAttribute` with properly encoded values instead of string concatenation.
5. **Content Security Policy (CSP):** Disallow inline scripts (e.g., `script-src 'self'`) to reduce the impact of any filter bypass.
6. **Cookie hardening:** Set `HttpOnly`, `Secure`, and `SameSite` attributes on session cookies.
7. **Security testing:** Include case-variation and filter-bypass payloads in XSS test suites, and use a trusted, well-maintained sanitization library instead of custom blacklist filters.
