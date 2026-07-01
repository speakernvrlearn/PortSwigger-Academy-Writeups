
**Date:** June 30, 2026  
**Target:** PortSwigger Academy - JWT authentication bypass via flawed signature verification  
**Severity:** High - Privilege Escalation / Full Administrative Account Takeover (ATO)

## Vulnerability
The application backend parses claims from the JWT payload but completely skips cryptographic signature verification.

## PoC
1. Log in as `wiener:peter`. Intercept traffic and send a valid `GET` request to **Repeater**.
2. Go to the **JSON Web Token** tab in the `Request` panel.
3. Change the payload claim: `"sub": "wiener"` -> `"sub": "administrator"`.
4. Copy the modified token from the `Raw` view (do NOT click `Sign`, leave the original signature intact).
5. Inject the forged token into the browser cookies via **F12** -> **Storage/Cookies**.
6. Navigate to `/admin` and delete user `carlos`.
