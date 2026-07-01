
**Date:** July 1, 2026  
**Target:** PortSwigger Academy - JWT authentication bypass via jwk header injection  
**Severity:** High - Privilege Escalation / Administrative Account Takeover (ATO)  
**Classification:** Intermediate (Practitioner)

---

## 1. Vulnerability Description
The target application utilizes JSON Web Tokens (JWT) signed with the asymmetric `RS256` algorithm for session management. A critical flaw exists in the signature verification architecture: the server implicitly trusts the optional `"jwk"` (JSON Web Key) parameter embedded directly within the user-controlled token header. 

Instead of validating incoming signatures against a strict internal whitelist of trusted organizational keys, the server extracts whatever public key is provided by the client, using it to verify the token's integrity. This allows an attacker to inject an arbitrary self-generated public key into the header and sign the forged token with their own private key, fully bypassing authentication.

---

## 2. Steps to Reproduce (PoC)

1. **Traffic Capture:** Logged into the platform using standard test credentials (`wiener:peter`) and intercepted the session traffic. Sent the authenticated `GET` request containing the session cookie to **Burp Suite Repeater**.
2. **Payload Modification:** Inside the **JSON Web Token** panel, accessed the token's `Payload` and successfully escalated privileges by changing the identity tracking claim `"sub": "wiener"` -> `"sub": "administrator"`.
3. **Key Pair Generation:** Navigated to the global **JWT Editor Keys** tab in Burp Suite and generated a clean asymmetric key pair by clicking **New RSA Key** (Format: `JWK`, Key Size: `2048` bits). The newly generated key was successfully saved into the tool's local cryptographic memory.
4. **Vulnerability Conceptualization:** Analyzed the initial token header and noted the absence of the `"jwk"` parameter. Recognized that the application backend must be forced into trusting an external key by manually injecting the parameter.
5. **JWK Header Injection Attack:** Returned to the **Repeater** module. Under the active JWT tab, selected **Attack** -> **Embedded JWK**, selected the newly generated custom RSA key, and clicked OK.
6. **Under the Hood Mechanics:** Burp Suite automatically modified the token by embedding the attacker's custom *Public Key* directly into the header block under the `"jwk"` claim, while simultaneously re-signing the entire forged payload using the attacker's custom *Private Key*.
7. **Exploitation:** Executed the request directly within the Repeater module by altering the endpoint route to the administrative portal (`GET /admin HTTP/2`). 
8. **Final Compromise:** The server successfully validated the token using the injected key, granting full administrator access. Completed the exploitation phase by executing the final action: deleting the user account `carlos`.

---

## 3. Impact
An external attacker can completely forge valid administrative sessions at will, leading to unauthorized access to the application's root dashboard. This enables full data modification, user account deletion, and total compromise of system business logic.

---

## 4. Remediation
* **Implement Strict Key Whitelisting:** Configure the backend token verification engine to strictly use a hardcoded or securely managed server-side database of trusted public keys.
* **Ignore Client-Side JWK Parameters:** Ensure the server-side JWT parsing library explicitly rejects or completely ignores any `"jwk"` or alternative key-embedding parameters passed inside the client's token header.
