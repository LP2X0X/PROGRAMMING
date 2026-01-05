---
tags: 
 - web
 - term
 - distinguish
 - fundamental
---

---
### Prerequisite
- [[Routing]]

---

## ⚡ Client-side vs Server-side Routing

### 1. **Server-side Routing (Traditional Websites)**

- Each time you click a link → **Browser sends request to the server**.
    
- Server processes the request, generates HTML, and sends a **new page** back.
    
- The browser refreshes to show that new page.
    
- Example: **PHP, ASP.NET, Django apps**.
    

✅ Pros:

- Good for SEO (search engines easily read server-rendered pages).
    
- Initial load may be faster (server sends a fully built page).
    

❌ Cons:

- Every navigation reloads the page.
    
- Slower user experience.
    
- More load on the server.
    

---

### 2. **Client-side Routing (React, Angular, Vue, SPA frameworks)**

- Only **one HTML page** is loaded initially (`index.html`).
    
- Clicking a link doesn’t reload the page. Instead, JavaScript updates the view.
    
- The **URL changes** in the browser (so it looks like navigation), but React Router decides which component to render.
    

✅ Pros:

- Faster navigation (no full reload).
    
- Smoother user experience (feels like a native app).
    
- Less server load (server mostly just serves APIs, not full HTML pages).
    

❌ Cons:

- Initial load may be slower (JavaScript bundle must load).
    
- SEO can be tricky (though React has solutions like **Next.js** for SSR).
    
- Requires JavaScript to function (if disabled, routing breaks).
    

---

### 3. **Visual Analogy**

- **Server-side**: Imagine going to different stores for every product → you leave one store, walk to another, and re-enter.
    
- **Client-side**: Imagine staying inside a big shopping mall → you walk between sections (views/components) without leaving the building.
    

---

👉 In React, we usually use **client-side routing** with `react-router-dom`.  

👉 If we need SEO or pre-rendered pages, we use **server-side rendering (SSR)** with frameworks like **Next.js**.