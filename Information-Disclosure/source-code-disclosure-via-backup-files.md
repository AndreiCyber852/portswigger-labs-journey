# Lab: Source Code Disclosure via Backup Files

## Difficulty

🟢 Apprentice

---

## Objectives

### Lab Objective

Retrieve the application's source code from an exposed backup file and use it to solve the lab.

### Learning Objective

Understand how exposed backup files can disclose sensitive source code, configuration data, and credentials that attackers can use during reconnaissance or further exploitation.

---

## Tools Used

* Burp Suite Community Edition
* Burp Repeater
* Web Browser

---

## Solution

### Step 1 - Discover the Backup Directory

While browsing the application, manually navigate to the `/backup` directory.

The server exposes a directory listing containing a Java backup file:

```text
/backup/ProductTemplate.java.bak
```

<img width="1266" height="716" alt="Backup directory listing" src="https://github.com/user-attachments/assets/6eee3584-4ccf-46c6-a21f-8535085ee768" />

---

### Step 2 - Analyze the Exposed Source Code

Open the backup file and review its contents.

The exposed backup file contains the application's source code, including hardcoded PostgreSQL connection details such as the database host, username, and password.

<img width="1272" height="729" alt="Exposed source code with hardcoded credentials" src="https://github.com/user-attachments/assets/5af814dc-647b-42b5-ad61-925b9bf54a1b" />

---

## Finding

The backup file disclosed sensitive source code that should never be publicly accessible.

Key information exposed:

* PostgreSQL database connection details
* Database username
* Database password
* Internal application implementation details

An attacker could leverage this information to understand the application's internal implementation, compromise backend services, or use the exposed credentials during further attacks.

---

## Mitigation

* Remove backup files from production environments.
* Disable directory listing on web servers.
* Never store hardcoded credentials in source code.
* Restrict access to backup and development artifacts.
* Perform regular security reviews to identify exposed files.

---

## Key Takeaways

* Backup files may expose an application's source code.
* Directory listing can unintentionally reveal sensitive files.
* Hardcoded credentials significantly increase the impact of information disclosure.
* Always test for exposed backup files during web application assessments.

---

## 🎥 Full Walkthrough

The following recording demonstrates the complete exploitation process, from discovering the exposed backup directory to retrieving the source code and identifying the hardcoded database credentials used to solve the lab.

https://github.com/user-attachments/assets/595ce5f0-c311-4e00-9bef-959ca3ceee7d
