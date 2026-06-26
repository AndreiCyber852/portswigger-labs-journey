# Lab: Remote Code Execution via Web Shell Upload

## Difficulty

🟢 Apprentice

---

## Objectives

### Lab Objective

Exploit an unrestricted file upload vulnerability to upload a PHP web shell, execute commands on the server, retrieve the contents of `/home/carlos/secret`, and solve the lab.

### Learning Objective

Understand how unrestricted file uploads can lead to Remote Code Execution (RCE) when uploaded files are executed by the web server.

---

## Tools Used

* Burp Suite Community Edition
* Burp Repeater
* Web Browser

---

## Solution

### Step 1 - Identify the File Upload Function

Log in using the provided credentials:

```text
Username: wiener
Password: peter
```

Navigate to **My Account** and locate the avatar upload functionality.

The application allows users to upload profile images without validating the uploaded file.

https://github.com/user-attachments/assets/7609cb29-77e0-4a77-a9b1-eeead8232ff4

---

### Step 2 - Upload a PHP Web Shell

Create a simple PHP web shell:

```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

Upload the file using the avatar upload functionality.

Because the application performs no validation, the PHP file is stored on the server and becomes accessible through the web.

https://github.com/user-attachments/assets/96fd467a-2f3c-4ce3-9429-835f16083ef2

---

### Step 3 - Execute the Web Shell

Browse to the uploaded PHP file.

Instead of displaying an image, the server executes the PHP code and returns the contents of:

```text
/etc/passwd
```

Copy the returned.

https://github.com/user-attachments/assets/4709bb09-51d5-49de-9e07-6214679ca366

---

### Step 4 - Submit the Secret

Submit the recovered secret using the lab banner.

The lab is successfully solved.

https://github.com/user-attachments/assets/0547e5b3-f936-402b-a25a-385e5b85b368

---

## Why does this work?

The application allows arbitrary file uploads without validating the file type or preventing executable files from being stored inside the web root.

Since PHP files are executed by the web server, uploading a PHP script results in **Remote Code Execution (RCE)**.

---

## Finding

The application is vulnerable to **Unrestricted File Upload**, allowing attackers to upload executable PHP files.

**Impact:**

* Remote Code Execution (RCE)
* Arbitrary command execution
* Disclosure of sensitive files
* Potential full server compromise

An attacker could upload a malicious web shell and execute arbitrary server-side code.

---

## Mitigation

* Validate uploaded file types using a whitelist.
* Verify the file's MIME type and content, not just its extension.
* Store uploaded files outside the web root.
* Disable execution permissions in upload directories.
* Rename uploaded files using random generated names.

---

## Key Takeaways

* Unrestricted file uploads are one of the most dangerous web vulnerabilities.
* Never trust the file extension supplied by the user.
* Uploaded files should never be executable.
* File uploads should be stored outside the web-accessible directory whenever possible.

---
