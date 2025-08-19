---
tags:
  - css
  - note
  - distinguish
---

## ✅ 1. **Does the root font size vary?**

Yes, the `root` font size (`html { font-size: ... }`) can **vary depending on:**

|Source of variation|Description|
|---|---|
|✅ **Browser default**|Most browsers default to `16px`.|
|✅ **User settings**|Users can override default font size (e.g., in accessibility settings).|
|✅ **CSS overrides**|If you set `html { font-size: 62.5%; }`, it becomes `10px` (`62.5% of 16px`).|
|✅ **Media queries**|You can make font-size responsive with media queries.|

So yes, **`rem` units depend on the root font-size**, and it **can be variable** depending on these factors.

---

## ✅ 2. **Can I only estimate font size when using `rem` or `em`?**

Not quite — you **can still calculate it precisely**, _if_ you know the base:

---

### 🔎 `rem` (root em):

- Always relative to the `html` font-size.
    
- **If `html { font-size: 16px; }`, then:**
    
    - `1rem = 16px`
        
    - `2rem = 32px`
        
    - `0.875rem = 14px`
        
- So if you know the root font size, **you can convert `rem` to `px` exactly**.
    

---

### 🔎 `em`:

- Relative to the **parent element’s font size**.
    
- So it can **cascade** or **compound**, making it harder to track.
    
    - Example:
        
        - If parent font-size = `20px`, then `1em = 20px`.
            
        - If child has `1.2em`, that means `1.2 * 20px = 24px`.
            

---

### ✅ To match design (in px) using rem:

If you're working from a design in px (like from Figma), just:

1. Assume root font-size is `16px` unless overridden.
    
2. Convert `px → rem` using:
    
    ```
    px ÷ 16 = rem
    ```
    

> Example:  
> Figma shows `font-size: 24px` → In CSS: `1.5rem`

---

### 🧠 Tip: Use `html { font-size: 62.5%; }`

That sets the root size to **10px**, making math simpler:

```css
html {
  font-size: 62.5%; /* 10px base */
}
```

Now:

- `1.4rem = 14px`
    
- `2rem = 20px`
    
- `0.8rem = 8px`
    

---

## ✅ Summary

|Unit|Relative To|Precise?|Use Case|
|---|---|---|---|
|`px`|Absolute|✅|Fixed size|
|`rem`|Root font-size|✅ if root known|Global consistency|
|`em`|Parent font-size|✅ but trickier|Component-local scaling|