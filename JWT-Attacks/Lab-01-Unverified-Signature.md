
**Date:** June 30, 2026  
**Target:** PortSwigger Academy - JWT authentication bypass via flawed signature verification  
**Severity:** High - Privilege Escalation / Full Administrative Account Takeover (ATO)

---

## 1. Vulnerability Description
The target application implements a JSON Web Token (JWT) mechanism for session handling. While the server is configured to perform signature verification under normal circumstances, it implicitly trusts the `"alg"` (algorithm) parameter provided in the user-controlled token header. 

Due to a flawed implementation in the backend signature-parsing logic, the server accepts the deprecated and insecure `"none"` algorithm. When an attacker switches the algorithm to `"none"`, the server disables its internal cryptographic verification checks and blindly processes the modified payload data.

---

## 2. Steps to Reproduce (PoC)

1. **Initial Login:** Authenticated into the web application using the provided low-privilege test credentials (`wiener:peter`). The server issued a valid session JWT token.
2. **Traffic Binding:** Navigated to an arbitrary endpoint on the site to trigger a new HTTP request containing the live session token.
3. **Interception:** Captured the request using **Burp Suite Proxy**, reviewed it in the **HTTP history**, and sent it to the **Repeater** module.
4. **Accessing the Editor:** Opened the **JSON Web Token** extension tab inside the left `Request` panel to deconstruct the token.
5. **Payload Tampering:** Located the identity tracking claim `"sub"` (Subject) within the `Payload` section and modified the value from `"wiener"` to the administrative username: `"administrator"`.
6. **Header Manipulation (The Twist):** Navigated to the `Header` section and altered the signing algorithm parameter from asymmetric RSA to unsecured mode: `"alg": "RS256"` -> `"alg": "none"`.
7. **Signature Stripping:** Removed the signature block entirely since no cryptographic algorithm was active. The resulting forged token was compiled with a trailing period representing an empty signature field:
   `eyJraWQiOiJiYjkzMTE4OS03M2UyLTQ5MTItYTljZi01YzVmNDZjMGUwN2QiLCJhbGciOiJub25lIn0.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc4MjgzODI2MCwic3ViIjoiYWRtaW5pc3RyYXRvciJ9.`
8. **Session Injection:** Opened the Developer Tools (**F12**) in the live browser window, navigated to the **Storage/Cookies** tab, and replaced the original session value with the newly modified unsecured JWT.
9. **Privilege Escalation:** Directed the browser to the administrative endpoint (`/admin`) and refreshed the page.
10. **Exploitation:** Successfully accessed the administrator panel with elevated privileges and executed the goal action by deleting the user account `carlos`.

---

## 3. Impact
An unauthenticated attacker can forge arbitrary session tokens by abusing the server's blind compliance with the `"alg": "none"` header. This allows for trivial privilege escalation, leading to a complete compromise of administrative functionalities and unauthorized access to data management features.

---

## 4. Remediation
* **Restrict Allowed Algorithms:** Configure the server-side JWT parsing library to strictly reject any tokens utilizing the `"none"` algorithm option.
* **Whitelisting:** Enforce a strict whitelist of permitted cryptographic signing algorithms (e.g., exclusively allow `RS256`) and ensure the server completely ignores the client-specified `"alg"` header if it falls outside the permitted parameters.
