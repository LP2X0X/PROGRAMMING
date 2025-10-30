---
tags: 
 - react
 - router
 - note
---

### 🧩 Why do we need to use the `:global` pseudo-class for the `active` class when styling React Router `<NavLink>` components with CSS Modules?

When you use **CSS Modules**, class names are **scoped locally** — they get transformed into unique identifiers like:

```css
nav_link__2HgK3
```

So if you write:

```css
.active {
  color: red;
}
```

it becomes something like:

```css
.active__1zXf5 { color: red; }
```

Now, React Router’s `<NavLink>` automatically applies a **plain global class** called `active` (literally the string `"active"`) to the link when it’s matched — **not** your locally scoped `.active__...`.

So your CSS Module class `.active__1zXf5` never matches React Router’s global `active`, meaning the style doesn’t apply.

#### 🧭 The solution — using `:global()`

To tell CSS Modules _“don’t rename this selector, treat it as global”_, you wrap it in `:global(...)`.

For example:

```css
:global(.active) {
  color: red;
}
```

Now your CSS rule targets the exact global class `active` that React Router adds.

#### 💡 Alternative (without `:global`)

You can skip `:global()` if you use **your own local class** conditionally:

```jsx
<NavLink
  to="/pricing"
  className={({ isActive }) => (isActive ? styles.active : styles.link)}
>
  Pricing
</NavLink>
```

Then in CSS:

```css
.link {
  text-decoration: none;
}

.active {
  color: red;
  font-weight: bold;
}
```