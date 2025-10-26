---
tags: 
 - web
 - mechanic
 - http
 - request
---

## 🌐 What Is an HTTP Request?

An **HTTP request** is a message sent from a **client** (usually a web browser or frontend app) to a **server** asking for data or an action.

Example:

- You type `https://example.com` → your browser sends an HTTP request to the server hosting that site.
    
- In React, you might call `fetch("https://api.example.com/data")` → that sends an HTTP request programmatically.
    

---

## 🧱 Structure of an HTTP Request

An HTTP request has **four main parts**:

```
1️⃣ Request Line
2️⃣ Headers
3️⃣ Empty line
4️⃣ Body (optional)
```

Example (POST request):

```
POST /api/users HTTP/1.1
Host: example.com
Content-Type: application/json
Authorization: Bearer abc123

{
  "name": "Long",
  "age": 25
}
```

Let’s break it down 👇

---

### 1️⃣ **Request Line**

This line defines **what you want to do**.

**Format:**

```
<HTTP Method> <Request Target> <HTTP Version>
```

Example:

```
GET /products HTTP/1.1
```

- **Method** → The type of action (GET, POST, etc.)
    
- **Request Target** → The resource path (like `/products`)
    
- **HTTP Version** → Usually `HTTP/1.1` or `HTTP/2`
    

---

### 2️⃣ **Headers**

Headers provide **extra information** about the request — who’s sending it, what’s inside, etc.

Common request headers:

|Header|Description|Example|
|---|---|---|
|`Host`|Domain name of the server|`Host: example.com`|
|`User-Agent`|Info about client app/browser|`User-Agent: Mozilla/5.0`|
|`Content-Type`|Type of body data|`application/json`|
|`Accept`|Expected response format|`Accept: application/json`|
|`Authorization`|Auth token|`Authorization: Bearer abc123`|
|`Cookie`|Session cookies|`Cookie: sessionId=xyz`|

---

### 3️⃣ **Empty Line**

Separates headers from the body.

---

### 4️⃣ **Body (Optional)**

Contains data sent to the server — only used in methods like `POST`, `PUT`, or `PATCH`.

Examples:

- **JSON:** `{"name": "Long"}`
    
- **Form Data:** `name=Long&age=25`
    
- **File upload:** binary/multipart data
    

---

## ⚙️ Common HTTP Methods

|Method|Purpose|Has Body?|Safe?|Idempotent?|
|---|---|---|---|---|
|`GET`|Retrieve data|❌|✅|✅|
|`POST`|Create resource|✅|❌|❌|
|`PUT`|Replace resource|✅|❌|✅|
|`PATCH`|Update part of resource|✅|❌|❌|
|`DELETE`|Remove resource|❌|❌|✅|
|`HEAD`|Like GET but no body|❌|✅|✅|
|`OPTIONS`|Ask what methods are supported|❌|✅|✅|

> ✅ **Safe:** doesn’t change data  
> ✅ **Idempotent:** same request repeated = same result

---

## 🔁 HTTP Request Lifecycle (Step by Step)

1. **DNS Lookup** – Find the IP address of the domain (e.g. `example.com` → `93.184.216.34`)
    
2. **TCP Handshake** – Establish a connection between client and server (3-way handshake)
    
3. **TLS Handshake (if HTTPS)** – Encrypt the connection
    
4. **Send HTTP Request** – Client sends request line, headers, and body
    
5. **Server Processes Request** – Server reads data, performs logic, queries DB, etc.
    
6. **Send HTTP Response** – Server returns status line, headers, and optional body
    
7. **Client Receives Response** – Browser or frontend handles the response (renders, stores, etc.)
    

---

## 📬 Example Response (for reference)

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 52

{
  "id": 1,
  "name": "Long",
  "role": "Developer"
}
```

---

## 🧰 In React / JS (Practical Example)

Using the Fetch API:

```js
fetch("https://api.example.com/users", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    Authorization: "Bearer abc123",
  },
  body: JSON.stringify({ name: "Long" }),
})
  .then((res) => res.json())
  .then((data) => console.log(data));
```

This example sends:

- **Method:** POST
    
- **Headers:** `Content-Type`, `Authorization`
    
- **Body:** JSON `{ "name": "Long" }`
    

---

## 🔒 HTTPS vs HTTP

|Feature|HTTP|HTTPS|
|---|---|---|
|Encryption|❌ None|✅ Encrypted via TLS|
|Port|80|443|
|Security|Vulnerable|Secure (Recommended)|

---

## 🧭 Summary

|Component|Description|
|---|---|
|**Request Line**|Defines method, path, and protocol|
|**Headers**|Add metadata about the request|
|**Body**|(Optional) Contains data for POST/PUT|
|**Response**|Server sends back status, headers, and data|
|**Lifecycle**|DNS → TCP/TLS → Request → Response|