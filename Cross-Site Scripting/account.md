## Reflected XSS - Extended Function Filter

## Description
**Affected URL:** `https://account.pwnedbox.site/`

A reflected Cross-Site Scripting (XSS) vulnerability exists in the application's input form. The `fname` parameter HTML-encodes special characters, but the `lname` parameter does not, reflecting user input unencoded into the page response. The application appears to filter `<script>` tags and/or the literal keyword `confirm`, but these are bypassed using a `<body onpageshow>` event handler combined with JavaScript string concatenation (`'c'+'o'+'n'+'f'+'i'+'r'+'m'`) to construct the function name without using the blocked keyword directly.

This is a security issue because the injected payload runs with the same privileges as the legitimate site's own code, and inconsistent encoding between parameters combined with an incomplete keyword filter leaves the application exploitable.

## Steps to Reproduce
1. Open the application in a browser: `https://account.pwnedbox.site/`
2. Enter the payload `navi'"<>` into the First Name and Last Name fields and submit. Observe that the payload is reflected but HTML-encoded in `fname`, while it is reflected unencoded in `lname`.
3. Submit the following payload via the `lname` parameter (and `fname`, as the same payload was used in both)
4. Submit the request and observe that a `confirm(1)` dialog is displayed, confirming successful reflected XSS execution via the `lname` parameter.
5. This payload works because the `<body onpageshow>` event fires automatically on page load without needing a `<script>` tag, and the `confirm` keyword is reconstructed at runtime via string concatenation, bypassing any filter that only checks for the literal word `confirm` or `<script>`.
6. The injected `<base href>` tag additionally rewrites the page's base URL to an attacker-controlled domain, which could be used to hijack relative-path resources (scripts, links, forms) loaded afterward.

## POC

Step 1: Enter `navi'"<>` into the First Name and Last Name fields and submit. Confirm `fname` is encoded while `lname` reflects the payload raw.

<img width="956" height="499" alt="image" src="https://github.com/user-attachments/assets/f98719b3-1bc0-41c1-921b-5360fc4219e1" />

Step 2: Submit the payload via the URL- navi'"<><body onpageshow=/*${/*/;{/**/this['c'+'o'+'n'+'f'+'i'+'r'+'m'](1)}//><Base/Href=//xss.bypass/c/navi>

<img width="955" height="495" alt="image" src="https://github.com/user-attachments/assets/0b230782-1878-4be8-88c5-e2e0a7f9d5b4" />


**[Insert screenshot here]**

**Result:** The `onpageshow` event handler fires automatically and reconstructs the `confirm` function via string concatenation, displaying a `confirm(1)` dialog and confirming reflected XSS via the `lname` parameter.

## Impact
1. Attacker-controlled script executes in the victim's browser session.
2. Session cookies/tokens can be stolen if not `HttpOnly`.
3. Victim accounts can be taken over via session hijacking.
4. Injected `<base href>` can redirect relative resource loads to an attacker-controlled domain.
5. Victims can be redirected to malicious or attacker-controlled sites.

## Remediation
1. Apply consistent output encoding to all parameters, including `lname`.
2. Do not rely on keyword blacklisting (e.g., `confirm`, `<script>`); use proper HTML/attribute encoding instead.
3. Strip or block dangerous HTML elements and event handler attributes (`onpageshow`, `<base>`, etc.) server-side.
4. Enforce a strict CSP to block inline event handlers and restrict `base-uri` to `'self'`.
5. Set `HttpOnly`, `Secure`, and `SameSite` flags on session cookies.
