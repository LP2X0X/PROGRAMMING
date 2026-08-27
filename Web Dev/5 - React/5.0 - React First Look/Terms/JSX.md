---
tags:
  - react
  - term
  - fundamental
---

## What Is JSX

JSX (JavaScript XML) is a **syntax extension for JavaScript** that lets you write HTML-like markup inside JavaScript files. It is not valid JavaScript on its own — it gets **compiled to regular function calls** by a build tool (Babel/SWC).

```jsx
// JSX — what you write
const element = <h1 className="title">Hello, {name}</h1>;

// What it compiles to — what the browser actually runs
const element = React.createElement('h1', { className: 'title' }, `Hello, ${name}`);
```

## Key Rules

### 1. Expressions go in curly braces `{}`

```jsx
const price = 9.99;
return <p>Price: {price * 1.1}</p>;    // any JS expression works
```

### 2. Must return a single root element

```jsx
// Wrong — two sibling elements
return (
    <h1>Title</h1>
    <p>Body</p>
);

// Fix — wrap in a fragment
return (
    <>
        <h1>Title</h1>
        <p>Body</p>
    </>
);
```

`<>...</>` is a **Fragment** — groups elements without adding an extra DOM node.

### 3. HTML attributes use camelCase

| HTML | JSX |
|---|---|
| `class` | `className` |
| `for` | `htmlFor` |
| `onclick` | `onClick` |
| `tabindex` | `tabIndex` |

This is because `class` and `for` are reserved words in JavaScript.

### 4. Self-closing tags are required

```jsx
<img src="photo.jpg" />    // must self-close
<input type="text" />      // can't leave it open like HTML
<br />
```

### 5. Inline styles use objects, not strings

```jsx
// HTML
<div style="background-color: red; font-size: 16px">

// JSX — double braces: outer = expression, inner = object
<div style={{ backgroundColor: 'red', fontSize: '16px' }}>
```

## Conditional Rendering

```jsx
// Ternary
return <p>{isLoggedIn ? 'Welcome back' : 'Please sign in'}</p>;

// Short-circuit (render only if true)
return <div>{hasError && <span className="error">Something went wrong</span>}</div>;
```

## Rendering Lists

```jsx
const items = ['Apple', 'Banana', 'Cherry'];

return (
    <ul>
        {items.map((item, index) => (
            <li key={index}>{item}</li>
        ))}
    </ul>
);
```

Every list item needs a unique `key` prop so React can efficiently track which items changed.

## Components in JSX

JSX can reference other components — this is how you compose your UI:

```jsx
function App() {
    return (
        <div>
            <Header />
            <ProductList items={products} />
            <Footer />
        </div>
    );
}
```

Component names must start with an **uppercase letter** — lowercase is treated as a native HTML element.