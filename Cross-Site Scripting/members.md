## Stored XSS - Product Reviews (Flipkart Clone)

## Description
**Affected URL:** `https://members.pwnedbox.site/index.php`

A stored Cross-Site Scripting (XSS) vulnerability exists in the "Product Review" feature of the application. User input submitted through the Headline and Detailed Review Comments fields is stored server-side as a Customer Review and rendered back into the product page without output encoding. The application appears to filter the literal lowercase `<script>` tag, but this blacklist-based filter is case-sensitive and does not account for mixed-case variations, allowing the filter to be bypassed.

This is a security issue because the injected script is persisted with the product's review data and executes automatically for any user who opens that product's page, running with the same privileges as the legitimate site's own code.

## Steps to Reproduce
1. Register an account and log in at `https://members.pwnedbox.site/index.php`.
2. Navigate to a product and select the "Product Review" option.
3. Submit a review with the following values:
   - Headline: `navi1'"<><ScrIpt>confirm("navi1")</ScrIpt>`
   - Detailed Review Comments: `navi1'2"<><ScrIpt>confirm("navi2")</ScrIpt>`
4. Submit the review and observe it is stored as a Customer Review on the product page.
5. Visit the same product page and observe that the `confirm("navi2")` dialog from the Detailed Review Comments field fires automatically, confirming the stored payload executes.
6. Have another user/session visit the same product page and observe the dialog fires again, confirming the payload is persistently stored and executes for any visitor.

## POC

Step 1: Submit a product review with the payloads `navi1'"<><ScrIpt>confirm("navi1")</ScrIpt>` as the Headline and `navi1'2"<><ScrIpt>confirm("navi2")</ScrIpt>` as the Detailed Review Comments.

<img width="956" height="529" alt="image" src="https://github.com/user-attachments/assets/a0ea2edf-c429-4540-a3f0-2e2693a0bac2" />

Step 2: Revisit the product page and observe the stored payload executes automatically.

<img width="959" height="430" alt="image" src="https://github.com/user-attachments/assets/4b2799e1-78cb-4517-b246-88c2ef3fc406" />


**Result:** The case-sensitive `script` filter does not block the mixed-case `ScrIpt` tag; the unsanitized payload from the Detailed Review Comments field is stored server-side and executes whenever the product page is loaded, displaying a `confirm("navi2")` dialog, confirming stored XSS.

## Impact
1. Attacker-controlled script executes automatically for every user who views the product page.
2. Session cookies/tokens can be stolen if not `HttpOnly`.
3. Any user's account, including administrators, can be taken over via session hijacking.
4. Fake login forms can be injected to phish credentials from product page visitors.
5. The application can be defaced or used to spread malware to all users who view the product.

## Remediation
1. Do not rely on case-sensitive keyword blacklisting (`<script>`); use proper output encoding instead.
2. Apply output encoding to the Headline and Detailed Review Comments fields wherever they are rendered.
3. Sanitize and validate input server-side before storing it, rejecting HTML/script content.
4. Enforce a strict CSP to block inline script execution regardless of filter bypasses.
5. Set `HttpOnly`, `Secure`, and `SameSite` flags on session cookies.
