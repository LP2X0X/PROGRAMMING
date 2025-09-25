---
tags:
  - web
  - general
---

- An SPA (Single-page application) is a web app implementation that loads only a single web document, and then updates the body content of that single document via JavaScript APIs such as [Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) when different content is to be shown.

- This therefore allows users to use websites without loading whole new pages from the server, which can result in performance gains and a more dynamic experience, with some tradeoff disadvantages such as SEO, more effort required to maintain state, implement navigation, and do meaningful performance monitoring.

### 📝 Detailed Notes

1. **Single-Page Application (SPA)**
    
    - Unlike traditional multi-page apps (MPAs), SPAs load a **single HTML page** and dynamically update it with JavaScript.
        
    - This avoids full page reloads and makes the app feel fast and seamless.
        
2. **Routing in SPA**
    
    - Clicking a link changes the **URL in the browser**, but instead of fetching a new page from the server, the **React Router** decides which component to render.
        
    - This is called **client-side routing**.
        
3. **JavaScript & React’s Role**
    
    - React manipulates the **DOM** to swap components in and out.
        
    - This keeps the user interface in sync with the current URL.
        
4. **Data Loading**
    
    - While the UI is managed by the client, the app may still fetch **data from the server via APIs** (e.g., user profile, product list).
        
    - This allows the UI to update **without reloading the page**.
        
5. **Native App Feel**
    
    - Because transitions are smooth and there’s no reload, SPAs give a **mobile app–like experience** in the browser.
        
6. **React Router**
    
    - Handles:
        
        - Matching URL → Component
            
        - Updating the browser history (`pushState`)
            
        - Rendering the correct view
            

---

⚡ In short:  
A **SPA (like React)** runs mostly in the browser, uses **client-side routing** to switch between views, and only calls the **server for data (APIs)**, not for full pages.