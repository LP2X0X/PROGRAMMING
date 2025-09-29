---
tags: 
 - html
 - element
---

- `<time>` is a **semantic HTML element** that represents either:
    
    1. A **specific point in time** (date/time), or
        
    2. A **duration** (like "3h 20m").
        

It helps browsers, search engines, and assistive tech understand time-related content.

---

## 🔑 Attributes

### 1. `datetime` (optional but important)

- Machine-readable value (ISO 8601 format).
    
- Lets you display human-friendly text but keep a precise, parsable time for machines.
    

Formats it accepts:

- Date only: `"2025-09-27"`
    
- Date & time: `"2025-09-27T08:30"`
    
- Duration: `"PT3H20M"` (3 hours 20 minutes)
    

---

## 📝 Examples

### Example 1: Simple date

```html
<p>Event date: <time datetime="2025-09-27">September 27, 2025</time></p>
```

- Displays: `September 27, 2025`
    
- Machines read: `2025-09-27`
    

---

### Example 2: Date and time

```html
<time datetime="2025-09-27T08:30">Today at 8:30 AM</time>
```

- User sees: `Today at 8:30 AM`
    
- Machine knows: `2025-09-27 08:30`
    

---

### Example 3: Duration

```html
<time datetime="PT2H30M">2 hours 30 minutes</time>
```

- User sees: `2 hours 30 minutes`
    
- Machine reads it as ISO duration.
    

---

### Example 4: Publishing date for SEO

```html
<article>
  <h2>Breaking News</h2>
  <p>Published on <time datetime="2025-09-27">September 27, 2025</time></p>
</article>
```

- Search engines pick up the publish date from the `<time>` tag.
    

---

## ✅ When to use `<time>`

- Blog/article publish dates
    
- Event schedules
    
- Timestamps in comments or posts
    
- Durations (workouts, cooking recipes, media length)
    
- Timelines
    

---

## ❌ When _not_ to use it

- If the content isn’t time-related (e.g., `time flies when you’re having fun` → here "time" is just a word, not data).
    

---

👉 In short:

- `<time>` = **semantic way to represent dates, times, and durations**.
    
- Use `datetime` attribute for machine-readability, while the inner text is for humans.
    