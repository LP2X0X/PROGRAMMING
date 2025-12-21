---
tags: 
  - css
  - property
  - flex
  - container
  - fundamental
---

# **📖 Definition**

The gap property (and its longhands row-gap & column-gap) defines the spacing **between items** in CSS layout models that support it (Flexbox, Grid, Multi-column layout).

Unlike margin, it **doesn’t create extra space outside items**, only **between them**.

---

# **🖋️ Syntax**

```css
/* Shorthand */
gap: <row-gap> <column-gap>;

/* Longhand */
row-gap: <length | percentage>;
column-gap: <length | percentage>;
```

- row-gap: space between rows.
    
- column-gap: space between columns.
    
- If you give only one value to gap, it’s applied to both rows and columns.
    

---

# **💡 Example**

```css
.container {
  display: flex;
  gap: 20px; /* 20px between flex items horizontally */
}

.grid {
  display: grid;
  gap: 10px 30px; /* 10px between rows, 30px between columns */
}
```

---

# **🛠️ Tips & Best Practices**

- ✅ Use gap instead of margins for spacing in flex/grid → cleaner code, no negative margin hacks.
    
- ✅ gap is directional-aware:
    
    - In flex-direction: row, it applies horizontally.
        
    - In flex-direction: column, it applies vertically.
        
    
- ✅ Works with flex-wrap: wrap; — gaps appear between wrapped lines too.
    
- ⚠️ Browser support: widely supported in modern browsers (Flexbox gap supported since Chrome 84, Firefox 63, Safari 14).
    
- ⚠️ gap does not work for display: block or inline layouts. Only Grid, Flexbox, Multi-col.
    

---

# **📊 Quick Comparison: gap vs margin**

|**Feature**|gap|margin|
|---|---|---|
|Scope|Only between items|Between items + outside container|
|Direction|Controlled by flex/grid|Manual control needed|
|Collapse|Never collapses|Vertical margins can collapse|
|Ease of use|Cleaner & less code|Needs special handling|