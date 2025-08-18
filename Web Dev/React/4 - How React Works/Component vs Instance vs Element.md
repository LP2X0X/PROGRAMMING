---
tags: react, internal, advance
---

### Component
👉 **Description** of a piece of UI
👉 A component is a function that **returns React elements** (element tree), usually written as JSX
👉 “Blueprint” or “Template”


### Component Instance
👉 Instances are created when we “use” components
👉 React internally calls component functions.
👉 Actual **“physical” manifestation** of a component
👉 Has its own state and props
👉 Has a **lifecycle** (can “be born”, “live”, and “die”)

### React Element
👉 JSX is converted to React.createElement() **function calls**
👉 A React element is the **result of these function calls**
👉 Information necessary to create **DOM elements**

![[Pasted image 20250810185402.png|center|500]]

----

## Detail

Understanding the differences between components, component instances, React elements and DOM elements is a crucial part of mastering React.

![](https://miro.medium.com/v2/resize:fit:700/1*-oXTMyLFgV2TGUY_bT3wIA.png)

> This flow show how the component is used to render the UI.

Since those entities are intertwined in the process of building a user interface, they can be hard to differentiate. Hopefully, understanding all of them independently helps.

Here is a simple and hopefully enlightening guide clarifying the differences between the four, very different, aspects of building React apps.

### COMPONENT

React components are often defined as simple functions that return **JSX (JavaScript XML)**, which describes the structure of the user interface elements. This JSX represents the layout and content of the component, making it easy to read and maintain.

> _So a component is only a_ **_description_** _of a piece of the UI._

They define the structure and appearance of these elements, but they are not the **instances** themselves. Instances are created based on these templates when rendered in the application.

Here's a simple component Button  

```jsx
function Button(props) {  
  return <button>{props.label}</button>;  
}
```

Unlike instances, which have lifecycles representing different stages, **components do not have lifecycles**. Instead, they provide the foundational structure and logic that instances inherit and utilize during their lifecycle. Thus, components can be thought of as the DNA or building blocks from which instances are constructed.

### COMPONENT INSTANCE

Component instances are **concrete occurrences** of React components within the application. Each instance represents a specific occurrence or usage of a component in the user interface.

React calls the instances the number of times they will be rendered. For example, you can have one button component, like we saw above, but use three instances in order to render three buttons on an app.

Here are three Button component instances:
```jsx
function NavBar() {  
  return (  
    <div>  
      <Button />  
      <Button />  
      <Button />  
    </div>  
  );  
}
```

Each instances have their own **states** and **props**, which are properties passed down from parent components. These state and props determine the appearance and behaviour of the instance.

Unlike components, instances have **lifecycles** representing different stages such as **creation, updating, and destruction**. Instances can be created when needed and destroyed when they are no longer required, helping to manage memory and resources efficiently.

### REACT ELEMENT

When you write JSX code in your React application, it gets converted into React.createElement() function calls behind the scenes.

> _The results of React.createElement() function calls are React elements._

React elements are **simple JavaScript objects that** describe the structure and content of a component in the **virtual DOM**. They serve as _lightweight_ representations of the UI components, defining what should be rendered on the screen.

They exist within the **runtime environment** of a React application, managed by React to represent the structure of components in the virtual DOM. I cannot show you a code example since they are not directly visible in the code.

**React elements include all the necessary information needed to create corresponding DOM elements.** This information includes the type of the element (e.g., ‘div’, ‘span’), its attributes (props), and its children elements.

When React elements are created, React **reconciles** them with the actual DOM, updating it to reflect any changes. Those elements are then **converted to real DOM elements** and updated in the browser’s display accordingly.

So, it’s important to keep in mind that React elements are **not the actual DOM elements!** Instead, they serve as **blueprints** or **templates** for what will be rendered.

### DOM ELEMENT (HTML)

Those, you might already know well by now. They are the actual HTML elements that exist in the Document Object Model. They represent the visual and interactive parts of the user interface. They are the different elements you can see in your browser’s developper tools.

### CONCLUSION

- Components are reusable UI pieces.
- Component instances are specific occurrences of those components in the app.
- React elements are lightweight descriptions of components in the virtual DOM
- DOM elements are the actual rendered elements in the browser.

Each plays a distinct role in building and rendering the user interface in a React application.