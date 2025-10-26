---
tags: 
 - js
 - request
---

### Prerequisite

[[Origin]]

---

## 1. What is a Cross-Origin Request?

Any time a script running on one origin tries to request a resource from a **different origin**, that’s a **cross-origin request**.

Example:

```js
// Running on https://myapp.com
fetch("https://api.othersite.com/data")
```

This is cross-origin because `myapp.com` ≠ `othersite.com`.

---

## 2. Why Restrictions Exist

Browsers enforce the **Same-Origin Policy (SOP)** for security.  
This prevents malicious scripts from reading sensitive data from another site (like banking info).

Without restrictions, any site could load your `https://bank.com/account` page and read private data.

---

## 3. How Browsers Handle Cross-Origin Requests

- **Simple requests** (like GET, POST with certain headers) are often allowed but restricted in what response data you can read.
    
- **Non-simple requests** trigger a **preflight request** (an `OPTIONS` request the browser sends first) to ask the server:  
    _“Do you allow this origin to access you?”_
    

---

## 4. CORS (Cross-Origin Resource Sharing)

If a request is cross-origin, the browser always adds the `Origin` header to it.

For instance, if we request `https://anywhere.com/request` from `https://javascript.info/page`, the headers will look like:

```http
GET /request
Host: anywhere.com
Origin: https://javascript.info
...
```

As you can see, the `Origin` header contains exactly the origin (domain/protocol/port), without a path.

The server can inspect the `Origin` and, if it agrees to accept such a request, add a special header `Access-Control-Allow-Origin` to the response. That header should contain the allowed origin (in our case `https://javascript.info`), or a star `*`. Then the response is successful, otherwise it’s an error.

The browser plays the role of a trusted mediator here:

1. It ensures that the correct `Origin` is sent with a cross-origin request.
2. It checks for permitting `Access-Control-Allow-Origin` in the response, if it exists, then JavaScript is allowed to access the response, otherwise it fails with an error.

![[Pasted image 20251016152525.png|center|600]]

Here’s an example of a permissive server response:

```http
200 OK
Content-Type:text/html; charset=UTF-8
Access-Control-Allow-Origin: https://javascript.info
```

---

## 5. Response headers

For cross-origin request, by default JavaScript may only access so-called “safe” response headers:

- `Cache-Control`
- `Content-Language`
- `Content-Length`
- `Content-Type`
- `Expires`
- `Last-Modified`
- `Pragma`

Accessing any other response header causes an error.

To grant JavaScript access to any other response header, the server must send the `Access-Control-Expose-Headers` header. It contains a comma-separated list of unsafe header names that should be made accessible.

For example:

```http
200 OK
Content-Type:text/html; charset=UTF-8
Content-Length: 12345
Content-Encoding: gzip
API-Key: 2c9de507f2c54aa1
Access-Control-Allow-Origin: https://javascript.info
Access-Control-Expose-Headers: Content-Encoding,API-Key
```

With such an `Access-Control-Expose-Headers` header, the script is allowed to read the `Content-Encoding` and `API-Key` headers of the response.

---

✅ **Summary**

- Same-Origin Policy blocks scripts from freely accessing other domains.
    
- Cross-Origin Requests go through **CORS**, where the server explicitly grants permission using headers.
    
- Browsers enforce it, not JavaScript itself.
    