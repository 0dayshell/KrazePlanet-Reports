## Stored XSS - Events & Polls Activity Center

## Description
**Affected URL:** `https://articles.pwnedbox.site/index.php`

A stored Cross-Site Scripting (XSS) vulnerability exists in both the registration/username feature and the "Create Interactive Poll" feature of the application. The username submitted at registration is stored server-side without sanitization and rendered back unencoded in the navigation bar (account menu) and in the "Created by" field of any poll the user creates. Additionally, the Poll Question, Poll Description, and each Poll Option field also store and reflect user input unencoded. Multiple distinct payloads were injected across these fields, and each one fires independently, confirming several separate stored XSS injection points on the same page.

This is a security issue because the injected payloads are persisted in the account and poll data and execute automatically on every page where this content is displayed, for every visitor, running with the same privileges as the legitimate site's own code.

## Steps to Reproduce
1. Register an account at `https://articles.pwnedbox.site/index.php` using the payload `navi1'"<ImG sRC=x onError confirm(1)>` as the username, and log in.
2. Navigate to "Events & Polls" and click "Create Interactive Poll".
3. Fill in the Poll Question with `navi1'"<ScRiPT>confirm("navi1")</ScRiPT>`, the Poll Description with `navi1'"<ScRiPT>confirm("navi1")</ScRiPT>`, and each Poll Option with a distinct payload — `navi2'"<ScRiPT>confirm("navi2")</ScRiPT>`, `navi3'"<ScRiPT>confirm("navi3")</ScRiPT>`, `navi4'"<ScRiPT>confirm("navi4")</ScRiPT>`, `navi5'"<ScRiPT>confirm("navi5")</ScRiPT>` — then click "Publish Poll".
4. Observe that the username payload is reflected unencoded in the top-right account menu, rendering a broken image with an `onerror` handler.
5. Visit the site (or have another user visit it) and observe that the `confirm(1)` dialog fires automatically from the account menu, followed by each distinct `confirm("naviX")` dialog from the Poll Question, Description, and Options fields firing in sequence, confirming multiple stored XSS injection points execute for any visitor.

## POC

Step 1: Create a poll using distinct payloads in the Question, Description, and Option fields and submit the poll.

<img width="935" height="532" alt="image" src="https://github.com/user-attachments/assets/5bec5895-6815-4513-bd8f-3d8d14f2c48b" />

<img width="950" height="491" alt="image" src="https://github.com/user-attachments/assets/48fd1811-7752-4d04-abee-a24857e2aa2f" />

<img width="957" height="520" alt="image" src="https://github.com/user-attachments/assets/b4706bb1-321b-4d66-b067-10fb4cf8d344" />

<img width="959" height="524" alt="image" src="https://github.com/user-attachments/assets/62f16a54-2a97-4908-b09f-814ea1d67482" />

**Result:** The username is reflected unencoded in the navigation bar and in the "Created by" line of the published poll, firing a `confirm(1)` dialog automatically via the broken `<img onerror>` tag. Additionally, each distinct payload entered in the Poll Question, Description, and Options fields (`navi1` through `navi5`) fires its own `confirm("naviX")` dialog in sequence when any visitor loads the polls page, confirming multiple stored XSS injection points on the same page.

## Impact
1. Attacker-controlled script executes automatically for every user who views the polls page or account menu.
2. Session cookies/tokens can be stolen if not `HttpOnly`.
3. Any user's account, including administrators, can be taken over via session hijacking.
4. Fake login forms can be injected to phish credentials from site visitors.
5. The application can be defaced or used to spread malware across the site wherever this content is displayed.

## Remediation
1. Apply output encoding to the username, Poll Question, Description, and Options fields wherever they are rendered.
2. Sanitize and validate all these fields server-side, rejecting HTML tags and event handler attributes.
3. Restrict usernames to a safe allowlist of characters (e.g., alphanumeric, underscores).
4. Enforce a strict CSP to block inline event handlers like `onerror` and inline script execution.
5. Set `HttpOnly`, `Secure`, and `SameSite` flags on session cookies.
