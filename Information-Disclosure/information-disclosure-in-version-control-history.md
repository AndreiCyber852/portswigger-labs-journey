# Lab: Information Disclosure in Version Control History

## Difficulty

🟡 Practitioner

---

## Objectives

### Lab Objective

Retrieve sensitive information exposed through the application's version control history, obtain the administrator password, log in as the administrator, and delete the user `carlos`.

### Learning Objective

Understand how exposed version control repositories can disclose sensitive historical information such as credentials, source code, and configuration files that enable further compromise.

---

## Tools Used

* Burp Suite Community Edition
* Web Browser
* Git
* GitTools (`git-dumper`)

---

## Solution

### Step 1 - Discover the Exposed Git Repository

While browsing the application, identify that the **`.git`** directory is publicly accessible.

Navigate to:

```text
/.git
```

or

```text
/.git/HEAD
```

The server returns Git repository files instead of denying access, indicating that the application's version control repository has been exposed.

This allows an attacker to download the exposed Git repository and inspect previous commits for sensitive information such as credentials, source code, and configuration files.

https://github.com/user-attachments/assets/65cf4c88-7df3-49b6-ad09-47f7cd8816d7

---

### Step 2 - Recover the Administrator Password and Complete the Lab

After downloading the repository, inspect the Git commit history:

```bash
git log
```

Identify the commit that modified the administrator password and inspect its changes:

```bash
git show <commit-hash>
```

The previous administrator password is revealed in the commit history.

Use these credentials to log in as the administrator, access the administrator interface, and delete the user **carlos**.

https://github.com/user-attachments/assets/352ccec8-6bd1-41e9-a5fd-558d99835d3e

---

#### Why does this work?

Git stores the complete history of every committed change. Even if sensitive information is removed from the latest version of the application, previous commits still preserve that data. If the `.git` repository is publicly accessible, attackers can recover historical credentials and use them to compromise the application.

---

## Finding

The application exposed its Git version control repository, allowing attackers to retrieve the complete commit history and recover sensitive information that had been removed from the current version of the application.

**Key information exposed:**

* Exposed `.git` repository
* Complete Git commit history
* Administrator credentials stored in a previous commit
* Source code and application history

An attacker could leverage this information to recover historical credentials, authenticate as privileged users, and compromise the application.

---

## Mitigation

* Never expose the `.git` directory on production web servers.
* Remove version control metadata before deploying applications.
* Store credentials securely using environment variables or a secrets manager.
* Regularly scan web applications for exposed sensitive files and directories.

---

## Key Takeaways

* Exposed Git repositories can reveal an application's complete development history.
* Sensitive information removed from the latest version may still exist in previous commits.
* Never commit passwords, API keys, or other secrets to version control.
* Always verify that the `.git` directory is inaccessible in production environments.

---
