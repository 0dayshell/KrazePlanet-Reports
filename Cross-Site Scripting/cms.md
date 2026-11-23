## Stored XSS - Shopify

## Description
**Affected URL:** `https://cms.pwnedbox.site/`

A stored Cross-Site Scripting (XSS) vulnerability exists in the "Write Article" feature of the application. User input submitted through the article "Description" field is stored server-side and rendered using `innerHTML` on the "My Articles" page, as explicitly noted in the application's own UI ("Content is saved as-is and rendered with innerHTML on your articles page"). This results in any HTML/JavaScript entered in the Description field executing without sanitization or encoding.

This is a security issue because the injected payload is persisted with the article data and executes automatically whenever the My Articles page is loaded, running with the same privileges as the legitimate site's own code.

## Steps to Reproduce
1. Log in to `https://cms.pwnedbox.site/` using test credentials (liam@quill.io).
2. Click "Write" to create a new article.
3. Enter the payload `navi1'"<><imG sRc=x onError=confirm("navi1")>` in the Title field, `navi2'"<><imG sRc=x onError=confirm("navi2")>` in the Description field, `navi4'"<><imG sRc=x onError=confirm("navi4")>` in the Description's HTML field, and `navi3'"<><imG sRc=x onError=confirm` in the Tags field.
4. Click "Save".
5. Observe that upon returning to the "My Articles" page, a `confirm("navi4")` dialog is displayed, confirming the Description field's payload executed via `innerHTML` rendering.

## POC

Step 1: Create an article with the payload `navi4'"<><imG sRc=x onError=confirm("navi4")>` in the Description field.

<img width="899" height="500" alt="image" src="https://github.com/user-attachments/assets/5ed4f9bc-553d-43f1-a810-409c7adb8154" />

Step 2: Save the article and return to the "My Articles" page.

<img width="954" height="366" alt="image" src="https://github.com/user-attachments/assets/65019046-6561-4442-b38a-7aa2a0e1d19a" />

**Result:** The Description field's content is rendered via `innerHTML` without sanitization, executing the injected `<img onerror>` tag and displaying a `confirm("navi4")` dialog, confirming stored XSS.

## Impact
1. Attacker-controlled script executes automatically whenever the My Articles page is viewed.
2. Session cookies/tokens can be stolen if not `HttpOnly`.
3. The article author's account, or any other user/admin who views the articles page, can have their session hijacked.
4. Fake login forms can be injected to phish credentials from anyone viewing the articles page.
5. The application can be defaced or used to spread malware via the articles listing.

## Remediation
1. Avoid rendering user-controlled content with `innerHTML`; use `textContent` or a sanitization library (e.g., DOMPurify) if HTML formatting must be preserved.
2. Apply output encoding to the Description field wherever it is rendered.
3. Sanitize and validate input server-side before storing it, rejecting HTML/script content and event handler attributes.
4. Enforce a strict CSP to block inline event handlers like `onerror`.
5. Set `HttpOnly`, `Secure`, and `SameSite` flags on session cookies.
