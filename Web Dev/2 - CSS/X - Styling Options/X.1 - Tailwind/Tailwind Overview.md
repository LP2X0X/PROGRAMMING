---
tags: 
 - css
 - tailwind
 - styling
 - option
 - overview
---

**Tailwind CSS** is a "utility-first" CSS framework that provides a comprehensive catalog of low-level CSS classes ([[Atomic CSS]]) (like `flex`, `pt-4`, `text-center`, and `rotate-90`) that can be composed to build any design, directly in your markup.

Unlike component-based frameworks (like Bootstrap or Material UI) that give you pre-built buttons and navbars, Tailwind gives you the **atomic building blocks** to create your own components.

![[Pasted image 20251122091426.png]]

```ad-note
Tailwind uses a **mobile-first styling approach**.
```

---

## 🧠 Core Philosophy: "Utility-First"

In traditional CSS, you write a semantic class name (`.profile-card`) and then write a block of CSS properties in a separate file.

In Tailwind, you apply pre-defined utility classes directly to the HTML element.

### Comparison

**Traditional CSS:**

```html
<div class="chat-notification">
  <div class="chat-notification-logo-wrapper">
    <img class="chat-notification-logo" src="/img/logo.svg" alt="ChitChat Logo">
  </div>
  <div class="chat-notification-content">
    <h4 class="chat-notification-title">ChitChat</h4>
    <p class="chat-notification-message">You have a new message!</p>
  </div>
</div>

<style>
  .chat-notification { display: flex; max-width: 24rem; margin: 0 auto; padding: 1.5rem; border-radius: 0.5rem; background-color: #fff; box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04); }
  /* ...and so on for every child element */
</style>
```

**Tailwind CSS:**

HTML

```html
<div class="mx-auto flex max-w-sm items-center gap-x-4 rounded-xl bg-white p-6 shadow-lg">
  <div class="shrink-0">
    <img class="h-12 w-12" src="/img/logo.svg" alt="ChitChat Logo">
  </div>
  <div>
    <div class="text-xl font-medium text-black">ChitChat</div>
    <p class="text-slate-500">You have a new message!</p>
  </div>
</div>
```

---

## ⚙️ Key Mechanics

### 1. The JIT (Just-In-Time) Engine1

Tailwind does not ship a massive CSS file with every possible class to your browser.2 Instead, it watches your HTML/JSX files as you code. When you type `text-blue-500`, the compiler generates that specific CSS rule on the fly and injects it.

- **Result:** Your production CSS file is tiny (usually <10kb) because it only contains the classes you actually used.
    

### 2. Modifiers (Responsive & States)

Tailwind handles conditional styling using a prefix system. You don't write media queries manually; you prefix the utility.3

- **Hover/Focus:** `hover:bg-blue-700`, `focus:ring-2`.
    
- **Responsive:** `md:w-1/2` (Apply width 50% only on medium screens and up).4
    
- **Dark Mode:** `dark:bg-slate-900` (Apply this background only when dark mode is active).
    

Example:

```html
<button class="bg-blue-500 hover:bg-blue-700 w-full md:w-auto">
  Submit
</button>
```

### 3. Arbitrary Values

If you need a very specific value that isn't in the default theme, you can use square brackets:

- `w-[350px]` (Width exactly 350px)
    
- `bg-[#1da1f2]` (Specific hex color)
    

---

## 🛠 Configuration (`tailwind.config.js`)

Tailwind is designed to be customized.5 You define your "Design System" (colors, fonts, breakpoints) in a config file.6 Tailwind then generates utility classes based on _your_ rules.

```js
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        'brand-blue': '#123456', // Generates: .bg-brand-blue, .text-brand-blue
      },
      fontFamily: {
        sans: ['Graphik', 'sans-serif'],
      },
    }
  }
}
```

---

## ⚖️ Pros & Cons7

| **Pros**                  | **Description**                                                                                                             |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **No Naming Wars**        | You stop wasting energy thinking of class names like `.sidebar-inner-wrapper`.                                              |
| **No jumping**            | Rapid prototyping. You never leave your HTML file, so there's no context switching between files.                           |
| **Easier to get back on** | Make it easier to collaborate and come back to project                                                                      |
| **Design system**         | Many design decision has been provided                                                                                      |
| **Time saving**           | Things like responsive design is fast and easy                                                                              |
| **Performance**           | Production CSS is extremely small due to unused CSS purging.                                                                |
| **Consistency**           | You can't pick "random" values (like `margin: 13px`). You must pick from the design scale (`m-4`), ensuring UI consistency. |

| **Cons**           | **Description**                                                                              |
| ------------------ | -------------------------------------------------------------------------------------------- |
| **"Ugly" HTML**    | The markup becomes cluttered with long strings of class names.                               |
| **Learning Curve** | You have to memorize the syntax (e.g., knowing that `tracking-wide` means `letter-spacing`). |

![[Pasted image 20251122094133.png|center|700]]
