---
tags: 
 - css
 - tailwind
 - flex
---

A Tailwind class is built from **up to four parts**, combined into one compact utility.

```
[prefix:][utility-name]-[value][modifier]
```

Each part has a specific job:

---

# 1️⃣ **Prefix (Optional) — When or Where It Applies**

Prefixes add a **condition**, **state**, or **responsive breakpoint**.

### **Types of prefixes**

- **Responsive breakpoints:**  
    `sm:`, `md:`, `lg:`, `xl:`, `2xl:`  
    → applies at certain screen widths.
    
- **Pseudo-class states:**  
    `hover:`, `focus:`, `active:`, `visited:`, `disabled:`  
    → applies only in that user interaction state.
    
- **Dark mode:**  
    `dark:`
    
- **Group / Peer interaction:**  
    `group-hover:`, `peer-checked:`
    
- **RTL/LTR:**  
    `rtl:`, `ltr:`
    

### **Example**

```css
hover:bg-blue-500
md:text-xl
group-hover:opacity-100
```

👉 Prefix always ends with `:`.

---

# 2️⃣ **Utility Name — What You Want to Style**

This is the **category of styling**.

### **Examples**

- Layout: `flex`, `grid`, `block`, `hidden`
    
- Spacing: `p`, `m`, `gap`
    
- Typography: `text`, `font`, `leading`, `tracking`
    
- Color: `bg`, `text`, `border`
    
- Borders/Radii: `rounded`, `border-x`, `border`
    
- Effects: `shadow`, `ring`, `opacity`
    
- Sizing: `w`, `h`, `min-w`, `max-h`
    

### **Example**

```css
bg-red-500   → bg = background-color
text-xl      → text = font-size
p-4          → p = padding
flex-col     → flex = display: flex
```

---

# 3️⃣ **Value — The Specific Setting**

The value defines _how much_, _which color_, _which size_, etc.

### **Examples**

- Numeric scale: `1`, `2`, `3`, `4`, `8`, `12`
    
- Color shade: `red-500`, `blue-600`, `gray-200`
    
- Size keywords: `full`, `auto`, `fit`, `min`, `max`
    
- Font sizes: `sm`, `lg`, `xl`, `2xl`
    
- Radius: `md`, `lg`, `full`
    
- Shadows: `sm`, `md`, `lg`
    
- Ring size: `2`, `4`, `8`
    

### **Example**

```css
p-4           → padding: 1rem
text-2xl      → font-size: 1.5rem
bg-gray-100   → background-color: #f5f5f5
w-full        → width: 100%
```

---

# 4️⃣ **Modifier (Optional) — Extra Fine Control**

Modifiers enhance or customize the utility value.

### **Common modifiers**

- **Opacity modifier**
    
    ```css
    bg-red-500/50     → 50% opacity background color
    text-black/80     → 80% opacity text
    border-blue-600/30
    ```
    
- **Directional or variant modifiers**
    
    ```css
    border-x    → border-left & border-right
    border-y    → border-top & border-bottom
    flex-row-reverse
    divide-y-2
    ```
    
- **Arbitrary values**
    
    ```css
    w-[250px]
    bg-[#ff5733]
    top-[3.75rem]
    ```
    

### **Example**

```css
bg-blue-600/30  → 30% opacity
shadow-lg       → preset shadow
flex-row-reverse
```

---

# 🧠 **Putting It All Together**

```
hover:bg-blue-500/50
 ↑    ↑        ↑
 |    |        └─ modifier (opacity 50%)
 |    └────────── utility-name + value (bg-blue-500)
 └──────────────── prefix (hover)
```

Another example:

```
md:focus:ring-4
```

- `md:` → breakpoint prefix
    
- `focus:` → state prefix
    
- `ring` → utility
    
- `4` → value
    

---

# 🎉 Summary Cheat Format

|Part|What it does|Example|
|---|---|---|
|**Prefix**|When/where it applies|`hover:`, `md:`, `dark:`|
|**Utility Name**|What property you want to change|`bg`, `text`, `p`, `ring`|
|**Value**|Specific value from Tailwind scale|`4`, `red-500`, `xl`|
|**Modifier**|Fine control / opacity / direction|`/50`, `-reverse`, `[20px]`|