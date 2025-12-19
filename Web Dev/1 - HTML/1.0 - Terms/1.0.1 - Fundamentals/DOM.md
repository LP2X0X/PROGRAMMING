---
tags:
  - html
  - term
  - fundamental
---

# **🔍 What is the DOM?**

- The DOM (Document Object Model) is a programming interface provided by browsers that represents an HTML (or XML) document as a structured tree of objects.
</br>
- Think of it as a live representation of your webpage.
- It lets programming languages like JavaScript interact with and manipulate the document’s content, structure, and styling.
</br>
- As the browser parses HTML, it creates a JavaScript object for every element and section of text encountered. These objects are called [[Node|nodes]]—element nodes and text nodes, respectively.

---

# **🧱 DOM Structure: Tree of Nodes**

- When a web browser loads an HTML page, it creates a DOM tree with different types of nodes:
	- Document node (the root)
	- Element nodes (like , , , etc.)
	- Text nodes (text inside elements)
	- Attribute nodes (like class=“box”, id=“header”)
	- Comment nodes (like )
  
📄 Example HTML:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Sample</title>
  </head>
  <body>
    <h1>Hello, world!</h1>
    <p id="para">Welcome to the DOM.</p>
  </body>
</html>
```

🧩 Corresponding DOM tree:

- document
    
    - html
        
        - head
            
            - title → “Sample”
                
            
        - body
            
            - h1 → “Hello, world!”
                
            - p (id=“para”) → “Welcome to the DOM.”
                
            
        
    

---

# **🛠 Interacting with the DOM using JavaScript**


You can use JavaScript to:
  

✅ Select elements
✅ Modify content or structure
✅ Handle events
✅ Change styles


## **🧭 1. Selecting Elements**

```js
document.getElementById("para");       // Selects element by ID
document.querySelector("h1");          // Selects first <h1>
document.querySelectorAll("p");        // Selects all <p> tags
```

## **✍️ 2. Modifying Content**

```js
const p = document.getElementById("para");
p.textContent = "Updated text.";
p.innerHTML = "<strong>Bold text</strong>"; // Be careful with innerHTML (can inject HTML)
```

## **🎨 3. Changing Styles**

```js
p.style.color = "blue";
p.style.fontSize = "18px";
```

## **🎯 4. Adding Event Listeners**

```js
const h1 = document.querySelector("h1");
h1.addEventListener("click", () => {
  alert("You clicked the heading!");
});
```

## **➕ 5. Creating and Adding New Elements**

```js
const newDiv = document.createElement("div");
newDiv.textContent = "I was added dynamically!";
document.body.appendChild(newDiv);
```

---

# **🔄 Live Connection**

The DOM is “live,” meaning if the HTML changes (e.g., through JavaScript), the DOM reflects those changes immediately.

---

# **🧠 Behind the Scenes: Browser Flow**

1. HTML is loaded.
2. Browser parses HTML into a DOM tree.
3. JavaScript can now access and manipulate that tree.
4. CSS is applied, layout is computed, and the screen is rendered.

---

# **📌 Summary**

|**Concept**|**Purpose**|
|---|---|
|DOM|Object-based tree of HTML elements|
|Nodes|Individual parts (elements, text, etc.)|
|DOM API|JavaScript interface to read/write DOM|
|Real-time Update|Changes via JS immediately affect webpage|
