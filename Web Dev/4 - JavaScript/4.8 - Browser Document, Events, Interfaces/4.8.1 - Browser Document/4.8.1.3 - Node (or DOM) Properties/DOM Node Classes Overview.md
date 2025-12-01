---
tags: 
 - js
 - DOM
 - property
 - overview
---

- **DOM node classes** are the **JavaScript constructor functions (interfaces)** that define the _type_ and _behavior_ of DOM objects — but you **cannot directly instantiate them**.

> They describe **what a DOM node _is_**, not **how to create it**.

- Different DOM nodes may have different properties.
- Each DOM node belongs to the corresponding built-in class.
</br>
- The root of the hierarchy is [EventTarget](https://dom.spec.whatwg.org/#eventtarget), that is inherited by [Node](https://dom.spec.whatwg.org/#interface-node), and other DOM nodes inherit from it.

- Here’s the picture, explanations to follow:
![[Pasted image 20250919132923.png|center|600]]

The classes are:

- [EventTarget](https://dom.spec.whatwg.org/#eventtarget) – is the root “abstract” class for everything.
    
    Objects of that class are never created. It serves as a base, so that all DOM nodes support so-called “events”.
    
- [Node](https://dom.spec.whatwg.org/#interface-node) – is also an “abstract” class, serving as a base for DOM nodes.
    
    It provides the core tree functionality: `parentNode`, `nextSibling`, `childNodes` and so on (they are getters). Objects of `Node` class are never created. But there are other classes that inherit from it (and so inherit the `Node` functionality).
    
- [Document](https://dom.spec.whatwg.org/#interface-document), for historical reasons often inherited by `HTMLDocument` (though the latest spec doesn’t dictate it) – is a document as a whole.
    
    The `document` global object belongs exactly to this class. It serves as an entry point to the DOM.
    
- [CharacterData](https://dom.spec.whatwg.org/#interface-characterdata) – an “abstract” class, inherited by:
    
    - [Text](https://dom.spec.whatwg.org/#interface-text) – the class corresponding to a text inside elements, e.g. `Hello` in `<p>Hello</p>`.
    - [Comment](https://dom.spec.whatwg.org/#interface-comment) – the class for comments. They are not shown, but each comment becomes a member of DOM.
- [Element](https://dom.spec.whatwg.org/#interface-element) – is the base class for DOM elements.
    
    It provides element-level navigation like `nextElementSibling`, `children` and searching methods like `getElementsByTagName`, `querySelector`.
    
    A browser supports not only HTML, but also XML and SVG. So the `Element` class serves as a base for more specific classes: `SVGElement`, `XMLElement` (we don’t need them here) and `HTMLElement`.
    
- Finally, [HTMLElement](https://html.spec.whatwg.org/multipage/dom.html#htmlelement) is the basic class for all HTML elements. We’ll work with it most of the time.
    
    It is inherited by concrete HTML elements:
    
    - [HTMLInputElement](https://html.spec.whatwg.org/multipage/forms.html#htmlinputelement) – the class for `<input>` elements,
    - [HTMLBodyElement](https://html.spec.whatwg.org/multipage/semantics.html#htmlbodyelement) – the class for `<body>` elements,
    - [HTMLAnchorElement](https://html.spec.whatwg.org/multipage/semantics.html#htmlanchorelement) – the class for `<a>` elements,
    - …and so on.

There are many other tags with their own classes that may have specific properties and methods, while some elements, such as `<span>`, `<section>`, `<article>` do not have any specific properties, so they are instances of `HTMLElement` class.

So, the full set of properties and methods of a given node comes as the result of the chain of inheritance.

---

````ad-note
DOM elements also have additional properties, in particular those that depend on the class:

- `value` – the value for `<input>`, `<select>` and `<textarea>` (`HTMLInputElement`, `HTMLSelectElement`…).
- `href` – the “href” for `<a href="...">` (`HTMLAnchorElement`).
- `id` – the value of “id” attribute, for all elements (`HTMLElement`).
- …and much more…

For instance:
```markup
<input type="text" id="elem" value="value">

<script>
  alert(elem.type); // "text"
  alert(elem.id); // "elem"
  alert(elem.value); // value
</script>
```

Most standard HTML attributes have the corresponding DOM property, and we can access it like that.

If we want to know the full list of supported properties for a given class, we can find them in the specification. For instance, `HTMLInputElement` is documented at [https://html.spec.whatwg.org/#htmlinputelement](https://html.spec.whatwg.org/#htmlinputelement).

Or if we’d like to get them fast or are interested in a concrete browser specification – we can always output the element using `console.dir(elem)` and read the properties. Or explore “DOM properties” in the Elements tab of the browser developer tools.
````

---

```ad-example
Let’s consider the DOM object for an `<input>` element. It belongs to [HTMLInputElement](https://html.spec.whatwg.org/multipage/forms.html#htmlinputelement) class.

It gets properties and methods as a superposition of (listed in inheritance order):

- `HTMLInputElement` – this class provides input-specific properties,
- `HTMLElement` – it provides common HTML element methods (and getters/setters),
- `Element` – provides generic element methods,
- `Node` – provides common DOM node properties,
- `EventTarget` – gives the support for events (to be covered),
- …and finally it inherits from `Object`, so “plain object” methods like `hasOwnProperty` are also available.
  
As we can see, DOM nodes are regular JavaScript objects. They use prototype-based classes for inheritance.
```