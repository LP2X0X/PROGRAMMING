---
tags: 
 - css
 - tailwind
 - config
---

# 🌟 Tailwind CSS v4 — Configuration Inside CSS (Detailed Overview)

Tailwind v4 (aka **Tailwind v4 / Oxide version**) introduces a **zero-config** design where configuration happens _inside your CSS_ instead of a JS config file.

This is done using **CSS @rules** that Tailwind reads at build time.

---

# ✅ 1. The New Core Idea

In Tailwind v4:

- No `tailwind.config.js`
    
- No plugins section
    
- No theme.extend
    
- No content array
    

All configuration is done in a **single CSS file** using these special rules:

```
@import "tailwindcss";          // main import
@theme { ... }                  // customize theme tokens
@custom-variant { ... }         // define custom variants
@layer utilities | components   // write your own layers
@plugin "..."                   // load a plugin (optional)
```

---

# 🎨 2. The `@theme` Rule (Most Important)

`@theme` replaces:

- theme()
    
- theme.extend
    
- colors, spacing, fontSize, screens, etc.
    
- full Tailwind config file
    

### Example:

```css
@import "tailwindcss";

@theme {
  --color-brand: #4f46e5;
  --color-brand-2: #818cf8;

  --spacing-xxs: 2px;
  --spacing-xs: 4px;

  --font-display: "Inter", sans-serif;

  --breakpoint-sm: 640px;
  --breakpoint-md: 768px;
}
```

### Why CSS variables?

Tailwind v4 uses **CSS custom properties** as your design tokens.

So a utility like:

```
text-brand
bg-brand-2
p-xs
```

are now generated from variables inside `@theme`.

---

# 🔧 3. Screens & Breakpoints

```css
@theme {
  --breakpoint-sm: 640px;
  --breakpoint-md: 768px;
  --breakpoint-lg: 1024px;
}
```

Usage:

```
sm:bg-brand
md:text-lg
```

---

# 🧩 4. Custom Utilities via `@layer utilities`

You can still write custom utility classes:

```css
@layer utilities {
  .text-shadow {
    text-shadow: 0 2px 4px rgb(0 0 0 / 0.1);
  }
}
```

---

# 🧱 5. Custom Components via `@layer components`

```css
@layer components {
  .btn {
    @apply px-4 py-2 rounded bg-brand text-white font-semibold;
  }
}
```

---

# 🌀 6. Adding Plugins

Still possible:

```css
@plugin "tailwindcss-typography";
@plugin "tailwindcss-animate";
```

Plugins now hook into the CSS system instead of JS.

---

# 🔄 7. Custom Variants (like `supports-[]`, `motion-safe`, etc.)

You can define your own variants:

```css
@custom-variant dark (&:where(.dark, .dark *));
@custom-variant hocus (&:is(:hover, :focus));
```

Usage:

```
dark:bg-gray-900
hocus:text-brand
```

---

# 📦 8. Fully Custom Tokens (Colors, Spacing, Radius, etc.)

Tailwind v4 uses **semantic tokens**, not the old theme maps.

Examples:

```css
@theme {
  --radius-md: 8px;
  --color-primary: oklch(62% 0.2 260);
  --color-danger: oklch(55% 0.25 25);
  --font-body: "Roboto", sans-serif;
}
```

Usage:

```
rounded-md
bg-primary
text-danger
font-body
```

---

# 🏗 9. Example Full Tailwind v4 Config CSS File

```css
/* tailwind.css */
@import "tailwindcss";

/* THEME TOKENS */
@theme {
  --color-brand: #1d4ed8;
  --color-brand-2: #3b82f6;

  --font-sans: "Inter", sans-serif;

  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 16px;

  --radius-md: 0.375rem;

  --breakpoint-sm: 640px;
  --breakpoint-md: 768px;
}

/* CUSTOM VARIANTS */
@custom-variant hocus (&:is(:hover, :focus));
@custom-variant dark (&:where(.dark, .dark *));

/* CUSTOM UTILITIES */
@layer utilities {
  .content-auto {
    content-visibility: auto;
  }
}

/* CUSTOM COMPONENTS */
@layer components {
  .btn {
    @apply px-md py-sm rounded-md bg-brand text-white font-semibold;
  }
}
```

---

# 🧠 10. Key Differences From Tailwind v3

| Tailwind v3                     | Tailwind v4                                  |
| ------------------------------- | -------------------------------------------- |
| `tailwind.config.js`            | **no config file**                           |
| JS-based theme extend           | `@theme` with CSS variables                  |
| content[] scanning              | automatic + simplified                       |
| JS plugins                      | CSS plugins (`@plugin`)                      |
| named color scales (`blue-500`) | create anything via tokens (`--color-brand`) |
| breakpoints in theme            | breakpoints via CSS variables                |

---

# 🎯 Summary

**Tailwind v4 config in CSS =**

- `@theme` → central place for all tokens
    
- `@layer` → custom utilities & components
    
- `@plugin` → add plugins
    
- `@custom-variant` → create new variants
    
- No JS config
    
- Everything is token-based and uses CSS variables
    

This makes Tailwind v4 more **portable**, **framework-agnostic**, **faster**, and **simpler**.