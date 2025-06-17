---
tags: html, note, fundamental
---

A **nested layout technique** in HTML/CSS means placing layout containers inside each other to build more complex, responsive structures. Think of it like boxes inside boxes—each controlling part of the layout.

---

### **💡 Common Use: Outer container for overall layout + inner containers for sections**

Here’s a basic example using **Flexbox**:

```html
<div class="outer">
  <div class="header">Header</div>
  
  <div class="main-layout">
    <div class="sidebar">Sidebar</div>
    <div class="content">Main Content</div>
  </div>
  
  <div class="footer">Footer</div>
</div>
```

```css
.outer {
  display: flex;
  flex-direction: column;
  height: 100vh;
}

.header,
.footer {
  background: #ccc;
  padding: 10px;
  text-align: center;
}

.main-layout {
  display: flex;
  flex: 1;
}

.sidebar {
  width: 200px;
  background: #eee;
  padding: 10px;
}

.content {
  flex: 1;
  background: #f9f9f9;
  padding: 10px;
}
```

---

### **🧱 You can nest layouts like:**

- flex inside grid
    
- grid inside flex
    
- even combine them with media queries for responsiveness
    

---

### Type of nested layout
![[Pasted image 20250417100133.png|center]]

---

### **📦 Other examples:**

- Dashboard with sidebar + main + nested cards
    
- Blog post with nested comments
    
- Multi-column sections inside a single-page layout
    

  
