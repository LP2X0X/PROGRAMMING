---
tags:
  - react
  - term
  - fundamental
---

In React’s architecture, a **renderer** is a library that connects React’s core (the reconciler) to a specific platform. React itself doesn’t know about the DOM, iOS views, or Android views—it just figures out what the UI _should_ look like. The renderer is what takes that description and turns it into actual UI.

---

### **Examples of official React renderers**

- **ReactDOM**
    
    - The renderer for web apps.
        
    - Uses the browser’s **DOM API** (document.createElement, appendChild, setAttribute, etc.) to build and update UI.
        
    - What you import when you do:
        

```js
import ReactDOM from "react-dom/client";
ReactDOM.createRoot(document.getElementById("root")).render(<App />);
```

- **React Native**
    
    - The renderer for iOS/Android apps.
        
    - Instead of the DOM, it talks to native platforms through a bridge and creates **native views** (UIView on iOS, View on Android).
        
    - You still write React components, but they render as native UI instead of HTML.
        
    

### **Other renderers**

- **react-three-fiber** → Renders to WebGL (for 3D).
- **Ink** → Renders to the terminal.
- **React-pdf** → Renders to PDF.

---

👉 So when people say _“ReactDOM is the web renderer, React Native is the native renderer”_, they mean each one is an implementation of React that knows **how to draw UI for its platform**.
