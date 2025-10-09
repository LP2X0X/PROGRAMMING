---
tags: 
 - js
 - request
---

At the lowest level, a **request** is just a **message** sent from one computer (the client) to another (the server), asking it to do something.

In the **web context** (HTTP/HTTPS):

- The **client** = usually the browser (but can be `curl`, Postman, mobile app, etc.).
    
- The **server** = the machine hosting the resource (website, API).
    
- The **request** = a structured message (text) that follows the **HTTP protocol**.
    

---

## 🔹 Anatomy of an HTTP Request

A request has 3 main parts:

### 1. **Request Line**

Specifies:

- **Method** (what action to perform) → `GET`, `POST`, `PUT`, `DELETE`, etc.
    
- **Path** (what resource) → `/api/users`
    
- **Version** → `HTTP/1.1`, `HTTP/2`
    

Example:

```
GET /api/users HTTP/1.1
```

---

### 2. **Headers**

Metadata about the request. They are key–value pairs like:

- `Host: example.com`
    
- `User-Agent: Mozilla/5.0`
    
- `Accept: application/json`
    
- `Authorization: Bearer <token>`
    

They tell the server _how_ to interpret the request.

---

### 3. **Body (optional)**

- Only used for methods that send data (`POST`, `PUT`, `PATCH`).
    
- Example: JSON payload, form data, file upload.
    

Example:

```
{
  "name": "Long",
  "age": 25
}
```

---

## 🔹 Example of a Full HTTP Request

```http
POST /api/users HTTP/1.1
Host: example.com
Content-Type: application/json
Authorization: Bearer abc123

{
  "name": "Long",
  "role": "admin"
}
```

---

## 🔹 Fundamentally

So **fundamentally**:

- A **request** = structured message from client → server.
    
- Contains: _action_ (method), _target_ (URL/path), _metadata_ (headers), and optional _data_ (body).
    
- Server processes it and replies with a **response** (another structured message with status + data).
    

---

## 🔹 Analogy

Imagine sending a letter:

- **Envelope** = headers (metadata: where from, where to, type).
    
- **Letter content** = body (the actual data).
    
- **Request line** = like writing “This is a complaint/request for info” at the top.
    

The post office (server) reads the envelope + content and decides how to respond.