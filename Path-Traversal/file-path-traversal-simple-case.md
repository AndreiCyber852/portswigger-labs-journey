# Lab: File Path Traversal, Simple Case

## Difficulty

🟢 Apprentice

---

## Tools Used

- Burp Suite Community Edition
- Firefox
- Kali Linux

---

## Vulnerability

- Path Traversal
- Arbitrary File Read (Local File Inclusion)

---

## Category

- Path Traversal

---

## OWASP

- **A01:2021 – Broken Access Control**

---

## CWE

- **CWE-22 – Path Traversal**

---

## Step 1 - Identify the Vulnerable Image Request

Before interacting with the application, configure **Burp Suite HTTP History** to reduce unnecessary traffic.

Hide common static resources such as CSS, fonts, icons, and JavaScript files while ensuring that **image requests remain visible**.

Reload the application and browse the product page until the images load.

Locate a request similar to:

```http
GET /image?filename=72.jpg HTTP/2
```

Send this request to **Burp Repeater**.

### Why?

The objective of this step is to identify the endpoint responsible for retrieving product images.

The application loads images using the **filename** parameter.

Unlike many requests that simply return HTML pages, this parameter directly determines which file the server will retrieve from the filesystem.

Since the application relies on user-controlled input to locate files, it immediately becomes a potential candidate for **Path Traversal** testing.

Filtering Burp Suite's HTTP History is an important skill.

If image requests are hidden or filtered out, the vulnerable endpoint may never be discovered, making exploitation impossible.

Once the correct request is identified, sending it to **Burp Repeater** allows us to safely modify and replay the request without interacting with the application again.

https://github.com/user-attachments/assets/13e317e9-bb4d-4c82-8002-d8b3b18f44fe

---

## Step 2 - Prepare the Request for Path Traversal Testing

Inside **Burp Repeater**, locate the original request:

```http
GET /image?filename=72.jpg HTTP/2
```

Notice that the application uses the **filename** parameter to determine which image should be returned.

At this point, replace only the value of the parameter while leaving the rest of the request unchanged.

### Why?

The vulnerable parameter is:

```text
filename=
```

Normally, it contains the name of an image such as:

```text
72.jpg
```

Internally, the application likely constructs a filesystem path similar to:

```text
/var/www/images/72.jpg
```

Because the filename is fully controlled by the user, it becomes a potential entry point for **Path Traversal**.

If the application does not properly validate this value, an attacker may replace the image name with directory traversal sequences to escape the intended directory.

At this stage, we have not exploited the vulnerability yet.

We are simply preparing the request that will be modified in the next step.

https://github.com/user-attachments/assets/8687b004-fbb7-4619-bf26-2c670e9ddcef

---

## Step 3 - Exploit the Path Traversal Vulnerability

Replace the original filename:

```text
72.jpg
```

with:

```text
../../../../../../../etc/passwd
```

The request now becomes:

```http
GET /image?filename=../../../../../../../etc/passwd HTTP/2
```

Click **Send**.

If the application is vulnerable, Burp Repeater returns the contents of:

```text
/etc/passwd
```

instead of an image.

The lab is now solved.

### Why does this work?

The vulnerable endpoint is:

```text
/image?filename=
```

The application expects the parameter to contain an image filename.

For example:

```text
72.jpg
```

Internally, the application probably generates a path similar to:

```text
/var/www/images/72.jpg
```

However, because the filename is not properly validated, replacing it with:

```text
../../../../../../../etc/passwd
```

causes Linux to resolve each directory traversal sequence.

Each:

```text
../
```

moves one directory higher.

Eventually, the application escapes the **images** directory and reaches the filesystem root.

The final resolved path becomes:

```text
/etc/passwd
```

instead of:

```text
/var/www/images/72.jpg
```

---

## Understanding Relative vs Absolute Paths

### Relative Path

A relative path starts from the application's current working directory.

Examples:

```text
images/72.jpg
```

```text
../../../../etc/passwd
```

Linux resolves these paths relative to the current directory.

---

### Absolute Path

An absolute path always starts from the filesystem root.

Examples:

```text
/var/www/images/72.jpg
```

```text
/etc/passwd
```

Because the path begins with:

```text
/
```

Linux knows the exact location of the requested file.

---

### Why `/etc/passwd`?

`/etc/passwd` exists on nearly every Linux system and is readable by all users.

It contains information about local user accounts, making it a safe and reliable file for confirming a successful Path Traversal vulnerability.

Penetration testers commonly use this file to verify exploitation because it does not modify the target system.

https://github.com/user-attachments/assets/eb097354-38c3-4304-80a1-ff33bb017c78

---

## Execution Flow

```text
Browser
      │
      ▼
filename=72.jpg
      │
      ▼
Application builds

/var/www/images/72.jpg

      │
      ▼
Attacker changes filename

../../../../../../../etc/passwd

      │
      ▼
Linux resolves directory traversal

      │
      ▼
/etc/passwd

      │
      ▼
Application returns the file contents
```

---

## What I Learned

- Product image requests can expose direct access to files stored on the server.
- Properly filtering Burp Suite HTTP History is essential for identifying vulnerable requests.
- Path Traversal works by escaping the intended directory using `../`.
- Linux resolves directory traversal sequences before opening the requested file.
- Understanding the difference between relative and absolute paths makes Path Traversal vulnerabilities much easier to understand.

**Video 4**

---

## Impact

Successful exploitation allows an attacker to:

- Read arbitrary files from the server.
- Access sensitive configuration files.
- Enumerate local users.
- Gather information useful for further attacks.

---

## Mitigation

Applications should:

- Validate filenames using an allowlist.
- Normalize paths before validation.
- Reject directory traversal sequences such as `../`.
- Restrict file access to a dedicated directory.
- Avoid exposing filesystem paths directly to user input.

---

## OWASP Mapping

- **A01:2021 – Broken Access Control**

---

## CWE Mapping

- **CWE-22 – Path Traversal**
