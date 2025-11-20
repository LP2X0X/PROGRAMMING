---
tags: 
 - js
 - post
 - request
 - pitfall
---

## **1. Forgetting to Set the Correct Headers**

When sending JSON, you must set:

```js
headers: {
  "Content-Type": "application/json",
}
```

If you forget this:

- Server may not parse your body
    
- `req.body` becomes `undefined`
    

---

## **2. Not Stringifying the Body**

Using fetch → body **must be a string** for JSON:

```js
body: JSON.stringify(data)
```

If you send `{ name: "John" }` directly:

- Server throws parse error
    
- Request silently fails depending on backend
    

---

## **3. Not Handling Errors Properly**

Many people check only network errors, but **fetch does NOT throw errors** on 4xx/5xx.

Bad:

```js
const res = await fetch(url);
const data = await res.json(); // crashes if server responds with HTML error page
```

Better:

```js
if (!res.ok) {
  const msg = await res.text();
  throw new Error(`Server error: ${msg}`);
}
```

---

## **4. CSRF Issues**

If you're working with:

- cookies
    
- authenticated areas
    
- frameworks like Django, Rails, Laravel
    

You may need CSRF tokens **even for POST**.

React devs often forget to include:

```js
credentials: "include"
```

---

## **5. Forgetting CORS Requirements**

Frontend → Backend on **different domain / port** must satisfy CORS.

Pitfalls:

- Accessing `localhost:5000` from `localhost:5173`
    
- POST blocked unless backend sets:
    

```
Access-Control-Allow-Origin: http://localhost:5173
Access-Control-Allow-Methods: POST
Access-Control-Allow-Headers: Content-Type
```

---

## **6. Sending Wrong Data Shape**

Backend expects:

```json
{ "email": "test@example.com" }
```

You send:

```json
{ "user": { "email": "test@example.com" } }
```

Or missing fields.  
Result: backend rejects → 400.

---

## **7. Not Awaiting the Promise**

Example mistake:

```js
const res = fetch("/api/create"); // NOT awaited
```

The request is still pending while your logic continues.  
You get bugs like:

- state updates before data returns
    
- navigation happens early
    

---

## **8. Race Conditions (Double Submits)**

User double-clicks submit → **two POST requests**.

Fix with:

- a “loading” state
    
- button disabled during request
    

```js
<button disabled={loading}>
```

---

## **9. Sending FormData Incorrectly**

When using `FormData`:

- DO NOT set `"Content-Type"` manually  
    Browser handles boundary automatically.
    

Wrong ❌:

```js
headers: { "Content-Type": "multipart/form-data" }
```

Correct ✔️:

```js
body: formData
```

---

## **10. Large Payload / Slow Network Issues**

Possible mistakes:

- Not setting timeout
    
- Not aborting slow requests
    

Use `AbortController`:

```js
const controller = new AbortController();
setTimeout(() => controller.abort(), 5000);
```

---

## **11. Not Handling JSON Parsing Errors**

If server sends non-JSON error (e.g. HTML page), this fails:

```js
await res.json();
```

Always check content-type or fallback to `.text()`.

---

## **12. Missing Authentication**

If backend expects headers:

```js
Authorization: Bearer <token>
```

and you forget:

- Request gets rejected
    
- Looks like a CORS problem sometimes (very confusing!)
    