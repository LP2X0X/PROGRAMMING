---
tags:
  - css
  - property
  - fundamental
---

The align-items property controls how **flex items are aligned on the cross axis** (perpendicular to the flex-direction).

---

### **📐 Syntax:**

```
.container {
  display: flex;
  align-items: <value>;
}
```

---

### **🧭 Cross Axis Depends On:**

- flex-direction: row → cross axis is **vertical**
    
- flex-direction: column → cross axis is **horizontal**
    

---

### **🔑 Common Values:**

|**Value**|**Description**|
|---|---|
|stretch|(default) Items stretch to fill container’s cross axis|
|flex-start|Items align to **start** of the cross axis|
|flex-end|Items align to **end** of the cross axis|
|center|Items align to **center** of the cross axis|
|baseline|Items align based on their **text baseline**|

  

---

### **💡 Example:**

```
<div class="container">
  <div class="item">One</div>
  <div class="item">Two</div>
</div>
```

```
.container {
  display: flex;
  height: 100px;
  align-items: center;
}
```

Result: All .items are vertically centered in the 100px tall container (assuming flex-direction: row).
