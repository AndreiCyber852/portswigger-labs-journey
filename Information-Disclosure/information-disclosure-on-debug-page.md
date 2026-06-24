# Lab: Information Disclosure on Debug Page

## Difficulty

🟢 Apprentice

---

## Objectives

### Lab Objective

Obtain and submit the `SECRET_KEY` environment variable that is exposed through the application's debug page.

### Learning Objective

Understand how publicly accessible debug pages can disclose sensitive information, such as environment variables, internal paths, and configuration details, which attackers can leverage during reconnaissance or further exploitation.

---

## Tools Used

* Burp Suite Community Edition
* Burp Repeater
* Web Browser

---

## Solution

### Step 1 - Inspect the HTTP Response

Navigate to the product page and inspect the HTTP response using **Burp Suite**.

While reviewing the response, notice that the HTML contains a hidden comment referencing a debug page.

<img width="945" height="488" alt="Hidden debug page reference" src="https://github.com/user-attachments/assets/704c92d3-4353-4c02-850c-72859017660c" />

---

### Step 2 - Access the Debug Page

The hidden comment references the following endpoint:

```text
/cgi-bin/phpinfo.php
```

Navigate to this endpoint. The page exposes the server's PHP configuration along with numerous environment variables.

Search for `SECRET_KEY`. The environment variable is publicly exposed and contains the value required to solve the lab.

<img width="1197" height="733" alt="SECRET_KEY exposed in phpinfo()" src="https://github.com/user-attachments/assets/de807b70-f5d7-4c19-a335-537f93ccffe2" />

---

## Finding

* **Exposed endpoint:** `/cgi-bin/phpinfo.php`
* **Sensitive information disclosed:** `SECRET_KEY` environment variable
* **Risk:** Publicly accessible debug pages can expose sensitive configuration, credentials, and environment variables that assist attackers during reconnaissance or lead to further compromise.

---

## Mitigation

* Disable debug pages in production environments.
* Restrict access to diagnostic and administrative endpoints.
* Never expose sensitive environment variables to unauthenticated users.
* Remove unnecessary debugging functionality before deploying applications to production.

---

## Key Takeaways

* Hidden comments may reveal sensitive endpoints.
* Debug pages should never be publicly accessible.
* Information disclosure vulnerabilities often provide valuable reconnaissance data that can facilitate further attacks.
* Always inspect HTML comments and unexpected application responses during web assessments.

---

## 🎥 Video Demonstration

The following recording demonstrates the complete exploitation process, from identifying the hidden debug page to retrieving the exposed `SECRET_KEY` and successfully solving the lab.

https://github.com/user-attachments/assets/a3beab3b-027c-432e-8218-17c94ebe0718















