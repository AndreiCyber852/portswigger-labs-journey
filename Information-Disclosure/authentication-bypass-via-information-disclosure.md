### Step 3 - Bypass the Authentication

Send the original request to **Burp Repeater** and add the following HTTP header:

```http
X-Custom-IP-Authorization: 127.0.0.1
```

The application trusts this header and treats the request as if it originated from the local machine (`127.0.0.1`), bypassing the IP-based access restriction.

As a result, the administrator interface becomes accessible.

#### Why does this work?

The `TRACE` method revealed the `X-Custom-IP-Authorization` header. The application incorrectly trusts this user-controlled header to determine the client's IP address. By setting its value to `127.0.0.1` (localhost), we impersonate a trusted local client and bypass the access control.

<img width="942" height="452" alt="Administrator interface accessed after bypassing authentication" src="PASTE_IMAGE_3_HERE" />

---

### Step 4 - Delete the Target User

After successfully accessing the administrator interface, locate the user **carlos** and select the **Delete** option.

The administrator interface is successfully accessed.

<img width="942" height="452" alt="Administrator interface" src="https://github.com/user-attachments/assets/f3b69ed6-2346-40ee-91c3-cf748fde12cd" />

After selecting **Delete**, the target user is removed from the application, successfully completing the lab.

<img width="942" height="452" alt="User carlos deleted" src="https://github.com/user-attachments/assets/bdf93131-dc8b-4d16-b462-098da0fc76ec" />

---

## Finding

The application disclosed a custom HTTP header that should never have been exposed to users.

**Key information exposed:**

* Custom HTTP header: `X-Custom-IP-Authorization`
* Trusted IP address: `127.0.0.1` (localhost)
* Internal access control implementation details

An attacker could leverage this information to spoof a trusted local request, bypass the IP-based access restriction, and gain unauthorized access to the administrator interface.

---

## Mitigation

* Disable the `TRACE` HTTP method unless it is explicitly required.
* Never trust user-controlled HTTP headers for authentication or access control decisions.
* Validate the client's IP address using trusted server-side mechanisms.
* Avoid exposing internal implementation details that could reveal security mechanisms.

---

## Key Takeaways

* Information disclosure can reveal internal implementation details that enable further attacks.
* The `TRACE` HTTP method may expose sensitive request information and should be disabled in production.
* Never trust user-controlled HTTP headers for authentication or authorization decisions.
* Always validate client identity using trusted server-side mechanisms.
