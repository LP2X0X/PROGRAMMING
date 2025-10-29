---
tags:
  - js
  - term
  - fundamental
---

In JavaScript, event types refer to the different kinds of interactions or occurrences that can be detected and responded to in the [[DOM|DOM]]. These include user actions like clicks, keyboard input, page loads, and more.

---

## **🧠 Common Categories of Event Types**

Here’s a categorized overview:

### **🖱️ Mouse Events**

|**Event**|**Description**|
|---|---|
|click|Fired when an element is clicked|
|dblclick|Double-click|
|mousedown|Button pressed|
|mouseup|Button released|
|mousemove|Mouse moved over element|
|mouseenter|Mouse entered element|
|mouseleave|Mouse left element|
|contextmenu|Right-click|

### **⌨️ Keyboard Events**

|**Event**|**Description**|
|---|---|
|keydown|Key pressed (fires continuously)|
|keyup|Key released|
|keypress|Key pressed (deprecated; use keydown)|

### **📱 Touch Events (Mobile)**

|**Event**|**Description**|
|---|---|
|touchstart|Finger touches screen|
|touchend|Finger lifted|
|touchmove|Finger moves across screen|
|touchcancel|Touch interrupted (e.g., by alert)|

### **🧭 Form Events**

|**Event**|**Description**|
|---|---|
|submit|Form submitted|
|change|Input value changed|
|input|Input value modified (real-time)|
|focus|Element focused|
|blur|Element lost focus|

### **📄 Window/Document Events**

|**Event**|**Description**|
|---|---|
|load|Page finished loading|
|unload|Page about to unload|
|resize|Window resized|
|scroll|Page or element scrolled|
|beforeunload|Prompt before leaving page|

### **🧩 Drag & Drop Events**

|**Event**|**Description**|
|---|---|
|dragstart|Drag begins|
|drag|Dragging|
|dragend|Drag ends|
|dragenter|Drag enters drop zone|
|dragover|Drag moves over drop zone|
|drop|Dropped in drop zone|
|dragleave|Drag leaves drop zone|

---

## **🧪 Example Usage**

```
const btn = document.getElementById("myButton");

btn.addEventListener("click", function (event) {
  console.log("Button clicked!", event);
});
```

---

## **🧠 How to Explore Event Types**

To see a full list of events and their properties:

- [MDN Web Docs – Event Reference](https://developer.mozilla.org/en-US/docs/Web/Events)
- Use your browser’s dev tools → console.dir(event) inside a listener