---
tags: html, term, fundamental
---

- Tag is the individual components inside angle brackets (<>).

---

```html
<p></p>
```

- `<p>` is the "opening" tag that tells the webpage that we are starting an HTML element. `</p>` is the "closing" tag (notice the `/`) that says to the browser, "hey, I'm done with the contents of this HTML element.
- There are many HTML tags available to you ([you can see them all here](https://developer.mozilla.org/en-US/docs/Web/HTML/Element)), but as a web developer starting out, you will only need a handful of them to create beautiful webpages and apps.

---

### Types of HTML tags

- [This page](https://developer.mozilla.org/en-US/docs/Web/HTML/Element) classifies HTML tags into many more categories, but here are the norms:
	1. Normal vs. "self closing" tags
	2. Container vs. Standalone tags

#### Normal vs. Self-Closing Tags

Most HTML tags have an "opening" and "closing" component to them. Here are some examples.

```html
<p></p>
<h1></h1>
<div></div>
<span></span>
<strong></strong>
```

- But some elements are what we call "self-closing". Here are common examples.

```html
<img />
<input />
<meta />
<link />
```

- You can nest elements within a normal HTML element while you cannot nest elements within a self-closing HTML element.

```html
<!-- Correct usage -->
<div>
  <p>some content</p>
</div>

<!-- INCORRECT usage (will not render) -->
<img <p>some content</p> />
```

#### Container vs. Standalone tags

- We know that we cannot "nest" or "embed" elements within a self-closing HTML element, but when should we be nesting HTML elements within the "normal" elements?

- For example, the following HTML **is valid**, but should we write it this way?

```html
<p>
  Some content
  <p>some nested content</p>
  <div>more nested content</div>
</p>
```

- **NO!!** While this HTML will render in the browser, it is improperly using the HTML tags. Some HTML tags such as `div`, `table`, `ol`, `ul`, and `body` are meant to have nested elements within them. Other HTML tags such as `h1`, `h2`, `h3`, `h4`, `h5`, `h6`, `p`, and `strong` are _not_ good as "containers" for other elements.