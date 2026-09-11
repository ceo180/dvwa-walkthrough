# Cross Site Request Forgery (CSRF)

Cross Site Request Forgery (CSRF) is an attack that forces an authenticated user to execute unwanted actions on a web application in which they are currently logged in. It exploits the fact that web browsers automatically include session cookies with requests.

---

## Security Level: Low

**The Exploit:**
The application's password change form relies solely on the user's session cookie for authentication and does not use any Anti-CSRF mechanisms. We can craft a malicious URL and trick the victim into clicking it while they are authenticated. 
**Payload Used:** `http://<target_ip>/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change`

<details>
<summary><b>📸 View Exploit & Source Code</b></summary>
<br>

![CSRF Low Exploit](images/csrf-low-exploit.png)
![CSRF Low Source](images/csrf-low-code.png)
</details>

**Code Analysis:**
The backend simply checks if the new passwords match and updates the database. It blindly trusts the GET request because the user's browser automatically attaches their valid session cookie, providing no way to verify if the user actually intended to make the request.

---

## Security Level: Medium

**The Exploit:**
The application now checks the `HTTP_REFERER` header to ensure the request originated from the target application's own server. We can bypass this by naming our malicious attacker domain or folder to include the target's hostname (e.g., hosting our exploit at `http://attacker-server.com/<target_ip>/exploit.html`).

<details>
<summary><b>📸 View Exploit & Source Code</b></summary>
<br>

![CSRF Medium Exploit](images/csrf-med-exploit.png)
![CSRF Medium Source](images/csrf-med-code.png)
</details>

**Code Analysis:**
The code uses `stripos( $_SERVER[ 'HTTP_REFERER' ] , $_SERVER[ 'SERVER_NAME' ])` to validate the referer. Because it only checks for the *presence* of the server name as a substring rather than enforcing an exact match, the validation is inherently flawed and easily spoofed.

---

## Security Level: High

**The Exploit:**
The application introduces an Anti-CSRF token (`user_token`) that changes per request. A standard CSRF attack fails because the attacker cannot guess the token. However, this can be bypassed by chaining it with a Cross-Site Scripting (XSS) vulnerability on the same domain to silently fetch a valid token via JavaScript before submitting the password change request.

<details>
<summary><b>📸 View Exploit & Source Code</b></summary>
<br>

![CSRF High Exploit](images/csrf-high-exploit.png)
![CSRF High Source](images/csrf-high-code.png)
</details>

**Code Analysis:**
The backend generates a token using `md5( uniqid( rand(), true ) )` and requires it to be submitted with the form. While strong against pure CSRF, the mechanism completely falls apart if the broader application has XSS flaws that allow DOM manipulation and token theft.

---

## Security Level: Impossible (Secure)

**The Remediation:**
The application is completely secure against CSRF and token-stealing XSS attacks for this specific function.

<details>
<summary><b>📸 View Secure Source Code</b></summary>
<br>

![CSRF Impossible Source](images/csrf-impossible-code.png)
</details>

**Code Analysis:**
The developer implemented the ultimate defense for sensitive account actions: **requiring the user's current password**. The backend validates the `password_current` parameter against the database. Even if an attacker steals an Anti-CSRF token, they cannot change the password without already knowing the victim's current credentials.
