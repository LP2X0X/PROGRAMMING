---
tags: 
 - css
 - tailwind
 - config
---

## **1. The Tailwind Config File**

Tailwind’s configuration lives in a file called `tailwind.config.js` (or `.ts` if using TypeScript). The config allows you to customize the default theme, add plugins, control what files Tailwind scans, and more.

A typical config looks like this (pre-v4):

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./src/**/*.{html,js,jsx,ts,tsx}", // files Tailwind should scan for class names
  ],
  theme: {
    extend: {}, // extend default theme values
    colors: {}, // override default colors
    spacing: {}, // override default spacing scale
    fontSize: {}, // override default font sizes
    screens: {}, // breakpoints
    // other theme properties...
  },
  plugins: [], // add Tailwind plugins
  corePlugins: {}, // enable/disable core utilities
  darkMode: 'media' | 'class', // configure dark mode
};
```

---

## **2. `content` (or `purge`)**

- Before Tailwind v3, this was called `purge`.
    
- This is **where you tell Tailwind which files to scan** for class names so it can remove unused CSS in production.
    

```js
content: ["./src/**/*.{js,ts,jsx,tsx,html}"]
```

- Wildcards like `**/*` mean **all files in all subdirectories**.
    
- Without this, Tailwind will generate all classes, making CSS huge.
    

---

## **3. `theme`**

The `theme` section controls all **design tokens**: colors, spacing, fonts, etc.

### **a) Default theme**

- Tailwind ships with a default set of values (colors, spacing, breakpoints).
    
- If you don’t touch the config, it uses them automatically.
    

```ad-warning
In Tailwind, adding values **inside `extend`** keeps the defaults and adds new ones, but adding them **directly in `theme`** overrides the entire section, removing all defaults.
```

### **b) `extend`**

- `extend` allows **adding new values** **without overwriting** the defaults.
    

```js
theme: {
  extend: {
    colors: {
      brand: "#1c64f2",
    },
    spacing: {
      128: "32rem",
    },
  },
}
```

- This keeps all default values and adds your custom ones.
    

### **c) Overriding defaults**

- If you define a property **outside of `extend`**, it replaces the default completely.
    

```js
theme: {
  colors: {
    primary: "#ff0000", // only this color exists, all defaults gone
  },
}
```

---

## **4. `screens` (breakpoints)**

- Tailwind provides default responsive breakpoints:
    

```js
screens: {
  sm: "640px",
  md: "768px",
  lg: "1024px",
  xl: "1280px",
  "2xl": "1536px",
}
```

- You can override or extend them in the config.
    

---

## **5. `plugins`**

- You can load **official or custom Tailwind plugins** here:
    

```js
plugins: [
  require('@tailwindcss/forms'),
  require('@tailwindcss/typography'),
]
```

- Plugins add **utilities or components**.
    

---

## **6. `corePlugins`**

- Allows enabling/disabling built-in Tailwind utilities.
    

```js
corePlugins: {
  preflight: false, // disables base styles
}
```

---

## **7. `darkMode`**

- Two modes:
    
    - `'media'` → uses `prefers-color-scheme` from the OS.
        
    - `'class'` → toggled via a `.dark` class.
        

```js
darkMode: 'class'
```

- Used like this in HTML:
    

```html
<html class="dark">
  <div class="bg-white dark:bg-gray-800">...</div>
</html>
```

---

## **8. Example Full Config (pre-v4)**

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./src/**/*.{js,ts,jsx,tsx,html}"
  ],
  theme: {
    extend: {
      colors: {
        brand: "#1c64f2",
      },
      spacing: {
        128: "32rem",
      },
    },
    fontFamily: {
      sans: ["Inter", "ui-sans-serif", "system-ui"],
    },
  },
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/typography'),
  ],
  darkMode: 'class',
};
```

---

### ✅ **Key Takeaways (pre-v4)**

1. `content` → what files to scan (used to be `purge`).
    
2. `theme` → all design tokens (extend or override).
    
3. `extend` → add without removing defaults.
    
4. `plugins` → add extra utilities/components.
    
5. `corePlugins` → enable/disable built-in utilities.
    
6. `darkMode` → configure dark mode behavior.
    
7. `screens` → responsive breakpoints.
    

---

### References

https://github.com/tailwindlabs/tailwindcss/blob/v3/stubs/config.full.js
