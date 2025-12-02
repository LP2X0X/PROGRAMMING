---
tags: 
 - js
 - DOM
 - html
 - attribute
 - note
---

* **HTML Attributes** — defined in the HTML (static). What you write inside the tag. 
* **DOM Properties** — exist on the DOM object in JavaScript (dynamic). What you access/change via JS. 

**In short:**

* Attributes reflect the HTML markup.
* Properties reflect the current state of the DOM object. 

---

## 🧩 Differences Between Attributes and Properties

| Attribute                                                        | Property                                                                                    |
| ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| Defined in HTML markup (e.g. `<div id="myDiv">`)                 | Defined on the DOM object (e.g. `element.id`)                                               |
| Names are **case-insensitive** (in HTML)                         | Names are **case-sensitive** (in JS)                                                        |
| Values are always **strings**                                    | Values can be **any type** (string, boolean, object, etc.)                                  |
| Represents the original or default configuration written in HTML | Represents the *current* state of the element (which can change via JS or user interaction) |

````ad-note
DOM properties are not always strings. For instance, the `input.checked` property (for checkboxes) is a boolean:

```html
<input id="input" type="checkbox" checked> checkbox

<script>
  alert(input.getAttribute('checked')); // the attribute value is: empty string
  alert(input.checked); // the property value is: true
</script>
```

There are other examples. The `style` attribute is a string, but the `style` property is an object:

```html
<div id="div" style="color:red;font-size:120%">Hello</div>

<script>
  // string
  alert(div.getAttribute('style')); // color:red;font-size:120%

  // object
  alert(div.style); // [object CSSStyleDeclaration]
  alert(div.style.color); // red
</script>
```

Most properties are strings though.

Quite rarely, even if a DOM property type is a string, it may differ from the attribute. For instance, the `href` DOM property is always a _full_ URL, even if the attribute contains a relative URL or just a `#hash`.

Here’s an example:

```html
<a id="a" href="#hello">link</a>
<script>
  // attribute
  alert(a.getAttribute('href')); // #hello

  // property
  alert(a.href ); // full URL in the form http://site.com/page#hello
</script>
```

If we need the value of `href` or any other attribute exactly as written in the HTML, we can use `getAttribute`.
````

---

## 🔄 Synchronization Between Attributes and Properties

* For many standard attributes, when the browser parses HTML, it creates corresponding DOM **properties** automatically. E.g. `<body id="page">` → `document.body.id === "page"`. 
* Changing an attribute (via `setAttribute`) often updates the corresponding property — and vice versa. 
* **But there are exceptions**: some properties do **not** sync back to attributes.

  * Example: setting `input.value = "new"` changes the property, but `getAttribute('value')` might still return the original string. 

```html
<input>

<script>
  let input = document.querySelector('input');

  // attribute => property
  input.setAttribute('value', 'text');
  alert(input.value); // text

  // NOT property => attribute
  input.value = 'newValue';
  alert(input.getAttribute('value')); // text (not updated!)
</script>
```

---

## 🛠️ How to Work with Attributes and Properties

* Use **properties** (dot-notation) when dealing with standard behaviors/state in JS. E.g. `element.id`, `input.checked`, `div.style.color`. 
* Use **attribute methods** when you need to access or manipulate exactly what’s in HTML (especially non-standard attributes):
	- `elem.hasAttribute(name)` – checks for existence.
	- `elem.getAttribute(name)` – gets the value.
	- `elem.setAttribute(name, value)` – sets the value.
	- `elem.removeAttribute(name)` – removes the attribute.
* For custom data attributes (e.g. `data-…`), use `elem.dataset`, which is the JS interface to those attributes. 

---

## ✅ When to Use What — Practical Guidance

* ✅ Use **properties** when you care about the element’s current interactive state or behavior (e.g. `input.value`, `checkbox.checked`, `elem.style`).
* ✅ Use **attributes** when you need to read/write what’s actually in the HTML — for example when working with custom attributes, when reading the original default values, or when setting something that should reflect in the HTML.
* ✅ Use `data-…` + `dataset` for safe custom data — avoids conflicts with future HTML standards. ([JavaScript.info][1])