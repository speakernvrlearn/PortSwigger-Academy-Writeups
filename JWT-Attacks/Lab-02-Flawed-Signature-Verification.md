
**Date:** June 30, 2026  
**Target:** PortSwigger Academy - JWT authentication bypass via unverified signature  
**Severity:** High - Full Administrative Account Takeover (ATO)

---

##  1. Vulnerability Description
The target web application utilizes JSON Web Tokens (JWT) for session management and user identification. However, a critical logical flaw exists within the backend implementation: the server processes and trusts data contained within the token's `Payload` but completely skips the validation of the cryptographic `Signature`. The server improperly handles tokens using a standalone `decode()` function rather than a secure `verify()` method.

This flaw allows an attacker to manipulate any parameters inside the Payload section (such as escalating privileges or altering usernames) and successfully authenticate with administrative rights using an invalid signature.

---

##  2. Steps to Reproduce (PoC)

1. **Initial Authentication:** Logged into the application using the standard provided test credentials `wiener:peter`. Upon successful login, the server generated a JWT and stored it client-side in the browser's storage (visible via the `Cookie: session=eyJ...` header).
2. **Token Binding:** Navigated to an arbitrary subpage on the website to force the browser to automatically attach the newly issued JWT token to a fresh HTTP request.
3. **Traffic Interception:** Intercepted the request using the **Burp Suite Proxy**, reviewed it in the **HTTP history**, and duplicated it to the **Repeater** module for isolated testing.
4. **Accessing the Editor:** Opened the specialized **JSON Web Token** extension tab inside the left `Request` panel.
5. **Payload Tampering:** Located the `"sub"` (Subject) identification claim within the `Payload` section. Modified the value from the standard test user `"wiener"` to the targeted administrative username — `"administrator"`.
6. **Token Export:** Since the target application backend does not enforce integrity checks on the signature, the newly modified token was copied directly from the `Raw` view without generating a new signature (the `Sign` button was intentionally left unclicked).
7. **Session Hijacking in Browser:** Returned to the live browser window, pressed **F12** to access Developer Tools, navigated to the **Storage/Application** tab, and located the **Cookies** section. Manually pasted the manipulated hacker JWT token over the original session value.
8. **Privilege Escalation:** Changed the URL endpoint in the address bar directly to the administrative panel interface (`/admin`) and refreshed the page.

---

##  3. Impact & Results
The browser automatically transmitted the forged token inside the request headers. The server successfully parsed the modified Payload, completely ignored the fact that the original signature was no longer mathematically valid, and granted full access to the administrative control panel. The attack was completed by executing a destructive action—deleting the user `carlos`—which successfully solved the laboratory assignment.

---

##  4. Remediation
* **Enforce Signature Verification:** Configure the backend application to never trust data within the Payload object until the entire token undergoes full cryptographic verification using the library's native `verify()` function tied to a strong server-side secret key.
