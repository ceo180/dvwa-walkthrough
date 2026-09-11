# SQL Injection (Blind)

Blind SQL Injection occurs when an application is vulnerable to SQL injection, but its HTTP responses do not contain the results of the relevant SQL query or any database errors. Instead of extracting data directly, the attacker must infer information by asking the database True/False questions (Boolean-based) or by instructing the database to pause before responding (Time-based).

---

## Security Level: Low

**The Exploit:**
When a valid ID is submitted, the application simply says, "User ID exists in the database." If invalid, it says, "User ID is MISSING from the database." Because no actual data is returned, we must inject Boolean logic to map the database. 
If we inject `1' AND 1=1 #`, the statement is True, and the application confirms the user exists. If we inject `1' AND 1=2 #`, the statement is False, and the application says the user is missing. We can use this True/False behavior to brute-force database names and passwords character by character.
**Payload Used (Boolean True):** `1' AND 1=1 #`
**Payload Used (Boolean False):** `1' AND 1=2 #`

<details>
<summary><b>📸 View Exploit & Source Code</b></summary>
<br>

**Execution Output (True vs False):**
![Blind SQLi Low Exploit](images/bsqli-low-exploit.png)

**Source Code:**
![Blind SQLi Low Source](images/bsqli-low-code.png)
</details>

**Code Analysis:**
Just like standard SQLi, the vulnerability stems from blindly concatenating user input:
```php
$id = $_GET[ 'id' ];
$query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
```
Even though the PHP script only returns a hardcoded text string rather than the database output, the query itself is still executing our injected logic.

---

## Security Level: Medium

**The Exploit:**
The application shifts to a POST request via a dropdown menu and sanitizes the input using `mysqli_real_escape_string()`. Because the query expects an integer rather than a string, we don't need quotes to break out. We can intercept the POST request using a proxy tool and inject an integer-based Boolean or Time-based payload.
**Time-based Payload Used:** `1 AND SLEEP(5) #`

<details>
<summary><b>📸 View Exploit & Source Code</b></summary>
<br>

**Execution Output (Burp Suite Intercept showing 5-second delay):**
![Blind SQLi Medium Exploit](images/bsqli-med-exploit.png)

**Source Code:**
![Blind SQLi Medium Source](images/bsqli-med-code.png)
</details>

**Code Analysis:**
The code attempts to escape quotes, but leaves the variable unquoted in the SQL statement:
```php
$id = mysqli_real_escape_string($GLOBALS["___mysqli_ston"], $id);
$query  = "SELECT first_name, last_name FROM users WHERE user_id = $id;";
```
Escaping quotes does nothing to prevent injection when the payload doesn't require quotes. Using a Time-based payload (`SLEEP(5)`) confirms the injection by forcing the web server to wait exactly 5 seconds before loading the page.

---

## Security Level: High

**The Exploit:**
The application moves the input mechanism to a separate window and relies on a session cookie, making automated exploitation via tools like SQLmap more complex. However, the backend query is still completely vulnerable to string manipulation. We can inject our time-based payload into the session input.
**Payload Used:** `1' AND SLEEP(5) #`

<details>
<summary><b>📸 View Exploit & Source Code</b></summary>
<br>

**Execution Output:**
![Blind SQLi High Exploit](images/bsqli-high-exploit.png)

**Source Code:**
![Blind SQLi High Source](images/bsqli-high-code.png)
</details>

**Code Analysis:**
Workflow obfuscation is used instead of actual sanitization:
```php
$id = $_COOKIE[ 'id' ];
$query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id' LIMIT 1;";
```
Reading the input from a cookie rather than a GET/POST parameter does not make the application secure. The database still executes the injected `SLEEP()` command.

---

## Security Level: Impossible (Secure)

**The Remediation:**
The application is fully secure against all forms of SQL Injection, including Blind SQLi.

<details>
<summary><b>📸 View Secure Source Code</b></summary>
<br>

![Blind SQLi Impossible Source](images/bsqli-impossible-code.png)
</details>

**Code Analysis:**
Just like the Impossible level of standard SQL Injection, the developer properly implemented **Prepared Statements** using PDO.
```php
$data = $db->prepare( 'SELECT first_name, last_name FROM users WHERE user_id = (:id) LIMIT 1;' );
$data->bindParam( ':id', $id, PDO::PARAM_INT );
$data->execute();
```
Because the SQL structure is compiled prior to the insertion of the user data, Boolean statements (`1=1`) and Time-based commands (`SLEEP`) are treated purely as a literal string of text, rather than executable database instructions.
