---
tags: 
 - css
 - tailwind
 - directive
---

The `@theme` directive is the core of the **CSS-driven configuration** introduced in Tailwind CSS v4.0. It allows you to define your design system variables (colors, spacing, fonts) directly in your CSS file rather than in a separate `tailwind.config.js` file.

In earlier versions of Tailwind, you used JavaScript to configure your theme. Now, the `@theme` directive acts as a bridge, turning standard CSS variables into Tailwind utility classes.

---

### 1. Basic Syntax

Inside your main CSS file, you open a `@theme` block. Any CSS variable defined inside this block is automatically picked up by Tailwind.

```css
@theme {
  /* This creates 'text-brand' and 'bg-brand' utilities */
  --color-brand: #3b82f6;
  
  /* This creates 'spacing-13' utilities */
  --spacing-13: 3.25rem;
  
  /* This creates 'font-display' utilities */
  --font-display: "Inter", sans-serif;
}
```

### 2. Extending vs. Overriding

One of the biggest shifts with the `@theme` directive is how it handles the "default" Tailwind theme.

- **To Extend:** Simply add new variables. Tailwind keeps all the default colors (like `blue-500`) and adds yours to the list.
    
- **To Override:** If you define a variable that already exists in the default theme (like `--color-blue-500`), your value will overwrite Tailwind's default.
    

> **Note:** If you want to **completely clear** the default theme and start from scratch (the equivalent of `theme: {}` in v3), you can use `@theme { ... }` but omit the `@import "tailwindcss";` at the top, or manually unset variables.

---

### 3. Key Naming Conventions

Tailwind uses a **prefix-based naming system** to know which utility a variable belongs to.

| **Variable Prefix** | **Tailwind Utility**         | **Example**                                               |
| ------------------- | ---------------------------- | --------------------------------------------------------- |
| `--color-*`         | `text-*`, `bg-*`, `border-*` | `--color-primary: #000;` $\rightarrow$ `bg-primary`       |
| `--spacing-*`       | `m-*`, `p-*`, `w-*`, `h-*`   | `--spacing-sm: 0.5rem;` $\rightarrow$ `p-sm`              |
| `--font-*`          | `font-*`                     | `--font-title: 'Roboto';` $\rightarrow$ `font-title`      |
| `--breakpoint-*`    | Responsive prefixes          | `--breakpoint-tablet: 640px;` $\rightarrow$ `tablet:flex` |
| `--radius-*`        | `rounded-*`                  | `--radius-lg: 1rem;` $\rightarrow$ `rounded-lg`           |

---

### 4. Referencing Theme Variables (The `theme()` function)

If you need to use one of your theme values elsewhere in your CSS (outside the `@theme` block), you use the `theme()` function.

```css
.custom-card {
  /* Grabs the value of --color-brand defined in @theme */
  background-color: theme('--color-brand');
  padding: theme('--spacing-4');
}
```

---

### 5. Why the `@theme` Directive is Better

1. **Single Source of Truth:** Your variables are standard CSS variables. You can use them in plain CSS, in inline styles, or via Tailwind utilities.
    
2. **No Context Switching:** You don't have to jump between a `.js` config file and your `.css` file.
    
3. **Real-time Dev Tools:** Because these are real CSS variables, you can modify them in the Browser Inspector (Chrome DevTools) and see the entire site update instantly without a re-compile.
    

---

### 6. Example: A Complete Design System

Here is how a modern Tailwind CSS file looks using the `@theme` directive:

```css
@import "tailwindcss";

@theme {
  /* Customizing colors */
  --color-brand-primary: oklch(0.6 0.18 250);
  --color-brand-secondary: oklch(0.4 0.12 200);

  /* Customizing breakpoints */
  --breakpoint-3xl: 1920px;

  /* Customizing animations */
  --animate-wiggle: wiggle 1s ease-in-out infinite;

  @keyframes wiggle {
    0%, 100% { transform: rotate(-3deg); }
    50% { transform: rotate(3deg); }
  }
}

@layer base {
  body {
    @apply bg-brand-primary text-white;
  }
}
```