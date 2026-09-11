**Command Injection Walkthrough**

Command Injection allows an attacker to execute arbitrary operating system commands on the host server via a vulnerable web application. This occurs when an application passes unsafe user-supplied data directly to a system shell.

---

## Security Level: Low

**The Exploit:** 
The application takes an IP address to ping and directly appends it to a system command without sanitization. By using command separators like `;` or `&&`, we can terminate the `ping` command and chain our own malicious commands.
**Payload Used:** `127.0.0.1 ; ls -la`

<details>
<summary><b>📸 View Exploit & Source Code</b></summary>
<br>

![Command Injection Low Exploit](images/cmd-inj-low-exploit.png)
![Command Injection Low Source](images/cmd-inj-low-code.png)
</details>

**Code Analysis:** 
The vulnerability exists because `shell_exec()` blindly concatenates and runs whatever the user submits, with zero input validation.

---

## Security Level: Medium

**The Exploit:** 
This level attempts to stop the previous attack by implementing a blacklist, removing the `&&` and `;` characters. However, the pipe operator `|` is not blacklisted, allowing us to bypass the filter and chain commands.
**Payload Used:** `127.0.0.1 | ls -la`

<details>
<summary><b>📸 View Exploit & Source Code</b></summary>
<br>

![Command Injection Medium Exploit](images/cmd-inj-med-exploit.png)
![Command Injection Medium Source](images/cmd-inj-med-code.png)
</details>

**Code Analysis:** 
The developer used `str_replace()` to blacklist specific operators. Blacklisting is inherently flawed because it leaves other valid command separators open for exploitation.

---

## Security Level: High

**The Exploit:** 
The blacklist is expanded to strip out almost all shell metacharacters, including `| ` (pipe followed by a space). By omitting the space after the pipe operator, we can bypass this strict filter.
**Payload Used:** `127.0.0.1|ls -la`

<details>
<summary><b>📸 View Exploit & Source Code</b></summary>
<br>

![Command Injection High Exploit](images/cmd-inj-high-exploit.png)
![Command Injection High Source](images/cmd-inj-high-code.png)
</details>

**Code Analysis:** 
A developer typo in the array (`'| ' => ''`) only targets the pipe symbol if followed by a space. Submitting a payload with no space executes successfully.

---

## Security Level: Impossible (Secure)

**The Remediation:** 
The application is finally secure against command injection. Blacklisting is abandoned in favor of strict input validation.

<details>
<summary><b>📸 View Secure Source Code</b></summary>
<br>

![Command Injection Impossible Source](images/cmd-inj-impossible-code.png)
</details>

**Code Analysis:** 
The IP address is broken down into four octets using `explode()`. The `is_numeric()` function checks every single octet to ensure it is strictly a number, completely rejecting all shell metacharacters.
