# Lab: Web Shell Upload via Content-Type Restriction Bypass

## Difficulty

🟢 Apprentice

---

## Objectives

### Lab Objective

Bypass the application's file upload validation by manipulating the `Content-Type` header, upload a PHP web shell, retrieve the contents of `/home/carlos/secret`, and solve the lab.

### Learning Objective

Understand why relying on user-controlled HTTP headers such as `Content-Type` for file validation is insecure and how attackers can exploit this weakness to achieve Remote Code Execution (RCE).

---

## Tools Used

* Burp Suite Community Edition
* Burp Repeater
* Burp Proxy
* Web Browser

---

## Solution

### Step 1 - Identify the File Upload Validation

Log in using the provided credentials:

```text
Username: wiener
Password: peter
```

Navigate to **My Account** and locate the avatar upload functionality.

Attempt to upload a PHP web shell.

The application rejects the upload because it validates the uploaded file's **Content-Type**.

https://github.com/user-attachments/assets/e16fca77-490f-4748-80b4-f8bb11aea2b0

---

### Step 2 - Bypass the Content-Type Validation

Intercept the upload request using **Burp Suite Proxy**.

Modify the following HTTP header:

```http
Content-Type: image/jpeg
```

instead of:

```http
Content-Type: application/x-php
```

Forward the modified request.

Because the application trusts the user-controlled `Content-Type` header, the PHP file is accepted and stored on the server.

https://github.com/user-attachments/assets/0abe2455-2515-47c0-bf92-c1b8c38eb3a6

---

### Step 3 - Execute the Web Shell

Browse to the uploaded PHP file.

The web server executes the PHP code instead of serving it as a file.

The script returns the contents of:

```text
/home/carlos/secret
```

Copy the secret and submit it using the lab banner.

The lab is successfully solved.

https://github.com/user-attachments/assets/d78a4ffd-3100-406e-b860-66b127c06cfe

---

## Why does this work?

The application validates uploaded files using the **Content-Type** HTTP header supplied by the client.

Since this header is completely under the attacker's control, it can easily be modified using Burp Suite.

Although the uploaded file is actually a PHP script, changing the `Content-Type` to `image/jpeg` causes the application to incorrectly trust the upload and store an executable PHP file on the server.

When the uploaded file is requested, the web server executes the PHP code, resulting in **Remote Code Execution (RCE)**.

---

## Finding

The application relies on the user-controlled **Content-Type** header to validate uploaded files.

**Impact:**

* Unrestricted upload of executable files
* Remote Code Execution (RCE)
* Disclosure of sensitive server files
* Potential full server compromise

An attacker could upload a malicious PHP web shell by spoofing the `Content-Type` header and execute arbitrary server-side code.

---

## Mitigation

* Never rely solely on the `Content-Type` header for file validation.
* Verify uploaded files using server-side content inspection.
* Restrict uploads to a whitelist of permitted file types.
* Store uploaded files outside the web root.
* Disable script execution within upload directories.
* Rename uploaded files using random generated names.

---

## Key Takeaways

* The `Content-Type` header is completely controlled by the client.
* Client-side validation should never be trusted.
* File uploads must be validated using server-side mechanisms.
* Uploaded files should never be executable by the web server.
* Weak file upload validation can quickly lead to Remote Code Execution.

---
