# Lab: Authentication Bypass via Information Disclosure

## Difficulty

🟢 Apprentice

---

## Objectives

### Lab Objective

Access the administrator interface by exploiting information disclosure to discover a custom HTTP header, then delete the user `carlos`.

### Learning Objective

Understand how seemingly harmless information disclosure can reveal implementation details that enable authentication bypass vulnerabilities.

---

## Tools Used

- Burp Suite Community Edition
- Burp Repeater
- Web Browser

---

## Solution

### Step 1 - Access the Admin Interface

Browse to the administrator panel:

```text
/admin
```

The application responds that the **admin interface is only available to local users**, indicating that access is restricted based on the client's IP address.

<img width="1229" height="714" alt="Admin interface restricted to local users" src="https://github.com/user-attachments/assets/3537e50f-5a75-4c34-81e5-f2ce8ccbd93c" />

<img width="943" height="523" alt="Unauthorized response from the admin interface" src="https://github.com/user-attachments/assets/a7659a10-f06a-4df3-9934-4c21b7840046" />

---

### Step 2 - Discover the Custom Header

Send a `TRACE` request to the `/admin` endpoint using **Burp Repeater**.

The `TRACE` HTTP method causes the web server to echo the original request back to the client. Although intended for debugging, it can unintentionally disclose internal implementation details when enabled.

In this lab, the reflected request reveals a custom HTTP header used by the front-end:

```text
X-Custom-IP-Authorization
```

The back-end trusts this header to determine the client's IP address. By supplying the value `127.0.0.1`, an attacker can impersonate a local request and bypass the IP-based access restriction protecting the administrator interface.

<img width="948" height="447" alt="TRACE response revealing the custom HTTP header" src="https://github.com/user-attachments/assets/c756874e-d7bf-497f-b28c-26b9fcdd322b" />

---

### Step 3 - Bypass the Authentication

Send the original request to **Burp Repeater** and add the following HTTP header:

```http
X-Custom-IP-Authorization: 127.0.0.1
```

The application **incorrectly trusts** this header and treats the request as if it originated from the local machine (`127.0.0.1`), bypassing the IP-based access restriction.

As a result, the administrator interface becomes accessible.

<img width="943" height="523" alt="image-63" src="https://github.com/user-attachments/assets/89942204-6fed-4a99-9745-0237a7ef1398" />

---

### Step 4 - Delete the Target User

After successfully accessing the administrator interface, locate the user **carlos** and select the **Delete** option.

The request is processed successfully, completing the lab.

<img width="942" height="452" alt="Deleting the user carlos" src="https://github.com/user-attachments/assets/f3b69ed6-2346-40ee-91c3-cf748fde12cd" />

<img width="942" height="452" alt="Lab solved after deleting the user carlos" src="https://github.com/user-attachments/assets/bdf93131-dc8b-4d16-b462-098da0fc76ec" />

---

## Finding

The application disclosed a custom HTTP header that should never have been exposed to users.

**Key information exposed:**

- Custom HTTP header: `X-Custom-IP-Authorization`
- Trusted IP address: `127.0.0.1` (localhost)
- Internal access control implementation details

An attacker could leverage this information to spoof a trusted local request, bypass the IP-based access restriction, and gain unauthorized access to the administrator interface.

---

## Mitigation

- Disable the `TRACE` HTTP method unless it is explicitly required.
- Never trust user-controlled HTTP headers for authentication or access control decisions.
- Validate the client's IP address using trusted server-side mechanisms.
- Avoid exposing internal implementation details that reveal security mechanisms.

---

## Key Takeaways

- Information disclosure can reveal implementation details that enable further attacks.
- The `TRACE` HTTP method may expose sensitive request information and should be disabled in production.
- Never trust user-controlled HTTP headers for authentication or authorization decisions.
- Always validate client identity using trusted server-side mechanisms.

---
