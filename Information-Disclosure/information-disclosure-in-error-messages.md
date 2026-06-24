# Lab: Information Disclosure in Error Messages

## Difficulty

🟢 Apprentice

---

## Lab Objective

Obtain the version number of the third-party framework that is disclosed through the application's verbose error messages.

## Learning Objective

Understand how verbose error messages can leak sensitive information such as framework versions that attackers can use for reconnaissance.

---

## Tools Used

- Burp Suite
- Browser

---

## Steps

### Step 1 - Intercept the request

Intercept the request using Burp Suite Proxy.


<img width="1276" height="771" alt="image-56" src="https://github.com/user-attachments/assets/db356c3d-f0da-4861-9443-889de4fbfad7" />




### Step 2 - Modify the request

Send the request to Repeater and change the HTTP:

```
productId=1
```

to

```
productId=example
```

<img width="1275" height="770" alt="image-57" src="https://github.com/user-attachments/assets/8eed0783-0f59-441b-9ac4-fc7f38139ea0" />


---

### Step 3 - Analyze the response

The server returned a verbose error message containing the version of the Apache Struts framework.



https://github.com/user-attachments/assets/3b87913c-9f5f-4853-98cc-367644cae5e8



---

## Flag / Solution

```
Apache Struts 2.3.31
```

---

## Key Takeaways

- Error messages should never expose software versions.
- Version disclosure helps attackers identify known vulnerabilities.
- Always inspect unexpected server responses during testing.
