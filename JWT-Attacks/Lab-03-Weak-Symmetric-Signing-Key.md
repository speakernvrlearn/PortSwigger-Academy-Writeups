
**Date:** June 30, 2026  
**Target:** PortSwigger Academy - JWT authentication bypass via weak symmetric signing key  
**Severity:** High - Offline Brute-Force / Full Administrative Account Takeover (ATO)  
**Classification:** Intermediate (Practitioner)

---

##  1. Vulnerability Description
The target application utilizes JSON Web Tokens (JWT) for session management and relies on the symmetric `HS256` (HMAC + SHA-256) algorithm to sign tokens. While the server properly enforces signature verification and rejects unsecured tokens (`alg: none`), it implements a highly insecure, weak, and predictable string as its secret signing key. 

Because JWT structure exposes both the header and the payload in plaintext format, an attacker can extract a valid token and perform a high-speed, local offline brute-force attack against the cryptographic signature using a dictionary of well-known secrets without alerting the server's defense mechanisms.

---

##  2. Steps to Reproduce (PoC)

1. **Session Capturing:** Authenticated into the target platform with the standard low-privileged account (`wiener:peter`) and intercepted the subsequent traffic via **Burp Suite Proxy** to extract the live session token from the cookie header.
2. **Offline Brute-Force Initial Failure & Correction:** Transferred the full JWT token into Kali Linux. Attempting to parse only the signature chunk into the auditing tool failed. Corrected the methodology by passing the **entire tripartite JWT string** to allow the utility to recalculate the mathematical hash properly.
3. **Executing Hashcat:** Ran a dictionary-based attack against the signature using the provided secrets list with the following optimized command:
   ```bash
   hashcat -a 0 -m 16500 eyJraWQiOiI2ZmEzMTViOS... jwt.secrets.list --show
   ```
4. **Key Discovery:** The offline brute-force attack successfully cracked the hash, exposing the weak server-side secret key: **`secret1`**.
5. **Key Injection in Burp Suite:** Opened the **JWT Editor Keys** tab in Burp Suite, clicked **New Symmetric Key**, switched the toggle to **`Specify secret`**, pasted the cracked value `secret1`, and hit *Generate* to save the target's signing key into the tool's memory.
6. **Payload Tampering & Re-Signing:** Returned to the **Repeater** module, accessed the **JSON Web Token** panel, and altered the identity claim from `"sub": "wiener"` to `"sub": "administrator"`. 
7. **Forging the Signature:** Clicked the **Sign** button, selected the newly injected `secret1` key, and explicitly selected the `Don't modify header` option to keep the original header tokens intact while cryptographically wrapping the forged payload using the authentic server key.
8. **Session Hijacking:** Copied the newly generated valid token from the `Raw` request panel, opened Browser Developer Tools (**F12**), injected the forged JWT into the cookie storage, and navigated straight to the `/admin` panel.
9. **Exploitation:** Gained absolute administrative privileges and successfully deleted the target user account `carlos`.

---

##  3. Impact
By abusing a weak symmetric key, an external attacker can completely compromise the token issuance integrity. This allows for total account takeover of any user on the platform, including system administrators, leading to unauthorized data destruction and full server management bypass.

---

##  4. Remediation
* **Implement Strong Secrets:** Replace the weak `secret1` signing key with a cryptographically secure, high-entropy, complex passphrase generated using an established random byte generator (minimum of 256 bits for HS256).
* **Environment Separation:** Ensure production environments never inherit default, testing, or hardcoded placeholder keys copied from open-source documentation or deployment templates.
