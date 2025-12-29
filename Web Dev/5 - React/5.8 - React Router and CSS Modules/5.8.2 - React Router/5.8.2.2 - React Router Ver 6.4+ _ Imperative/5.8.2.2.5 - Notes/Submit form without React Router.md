---
tags: 
 - react
 - router
 - note
 - submit
 - form
---

## 🌐 **How Form Submission Works Without React Router (Normal Browser Way)**

Before React, before React Router, before SPAs…

A form submit worked like this:

1. You click **Submit**
    
2. The browser sends an HTTP request to the server
    
3. The browser **reloads the whole page**
    
4. The server responds with a **new HTML page**
    
5. Browser replaces the old content with the new content
    

No JavaScript needed.

---

## 📌 **Basic Example (Traditional HTML Form)**

```html
<form action="/login" method="POST">
  <input name="username">
  <input name="password" type="password">
  <button type="submit">Login</button>
</form>
```

#### What happens when you click submit?

- Browser collects all input values
    
- Creates a **FormData**
    
- Converts it to `application/x-www-form-urlencoded` or `multipart/form-data`
    
- Sends it to the server
    
- Browser reloads to the new URL or replaces the current page
    

#### Request example:

```
POST /login
Content-Type: application/x-www-form-urlencoded

username=long&password=1234
```

---

## 🧭 **Full page reload is the default behavior**

This is the biggest difference from SPAs:

- ❌ No client-side state preserved
    
- ❌ UI resets
    
- ❌ Scroll resets
    
- ❌ App loses all JavaScript state
    
- ❌ Cannot prevent reload unless you write JavaScript manually
    

React Router fixes this.

---

## 💡 **How we handled form submit with JavaScript (Pre-React SPA)**

Before React Router, you would do:

```html
<form id="myForm">
  <input name="username">
  <button>Submit</button>
</form>

<script>
document.getElementById("myForm").addEventListener("submit", function(e) {
  e.preventDefault(); // stop reload

  const formData = new FormData(e.target);

  fetch("/api/login", {
    method: "POST",
    body: formData
  });
});
</script>
```

You had to handle:

- preventing reload
    
- reading form data
    
- calling fetch
    
- updating UI manually
    
- handling loading states
    
- showing errors
    
- redirecting manually
    

It was a lot of repetitive boilerplate.

---

## 🆚 **Normal Form vs React Router `<Form>`**

#### 🏛 **Normal form**

- Browser sends request
    
- Server returns HTML
    
- Browser reloads
    
- UI resets completely
    

#### 🚀 **React Router `<Form>`**

- Intercepts submit
    
- Runs an `action()` on the client
    
- Updates UI without reload
    
- Shows submitting/loading state
    
- Revalidates loaders
    
- Handles redirects on the client
    
- Keeps SPA state
    

---

## 🌀 Visual flow comparison

#### **Normal (traditional browser):**

```
Form → Browser → Server → HTML → Reload page
```

#### **React Router `<Form>`:**

```
Form → React Router → action() → update UI → revalidate data → no reload
```

---

## 🔥 Summary

#### **Normal form submission (non-React Router):**

- Sends request to server
    
- Full page reload
    
- Server decides what HTML to send back
    
- No SPA features
    

#### **React Router form submission:**

- No reload
    
- Data sent to `action()`
    
- SPA stays alive
    
- Built-in pending/loading state
    
- Auto revalidation
    
