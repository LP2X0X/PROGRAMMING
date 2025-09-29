---
tags: 
 - react
 - question
---

We don’t **have** to make the URL change when the GUI changes — but in web apps, we usually _want_ to. Here’s why:

---

### 🔑 Reasons URLs matter in SPAs (Single Page Apps like React)

1. **Shareability**
    
    - If the state of the app is only in memory (not reflected in the URL), you can’t copy the link and send it to someone.
        
    - Example: If you’re on `/dashboard/profile` and copy the URL, your friend can open the exact same screen.
        
2. **Back/Forward Navigation**
    
    - Browsers already give us Back/Forward buttons tied to the history stack.
        
    - If UI changes don’t affect the URL, the back button won’t work as users expect.
        
3. **Bookmarking**
    
    - Users can bookmark `/settings` or `/product/123` and come back later. Without a URL change, they’d just get dropped on the app’s home every time.
        
4. **Deep Linking**
    
    - If you open a notification or email with a link like `https://app.com/messages/42`, you go straight to the right place.
        
    - Without URLs tied to app state, this isn’t possible.
        
5. **SEO / Crawlers** (if public app)
    
    - Search engines and crawlers need stable URLs to index content.
        
6. **Consistency with Web Paradigm**
    
    - The web has always been URL-driven. Using URLs for navigation feels natural to users (and developers).
        

---

### ⚖️ When _not_ to change the URL

If the change is **purely visual or temporary state**, you don’t need to update the URL:

- Opening a dropdown
    
- Switching a tab inside a component (unless you want shareable state)
    
- Expanding/collapsing sections
    

---

👉 So, the rule of thumb is:

- **If the state matters for navigation, sharing, or revisiting → put it in the URL.**
    
- **If it’s just UI cosmetics → keep it in component state.**
    