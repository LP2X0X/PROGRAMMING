---
tags: 
 - css
 - function
 - overview
---

**CSS value functions** are statements that invoke special data processing or calculations to return a CSS value for a CSS property. CSS value functions represent more complex data types and they may take some input arguments to calculate the return value.

### 🔹 Categories of CSS Functions

#### 1. **Color functions**

- `rgb(255, 0, 0)` → red color.
    
- `rgba(255, 0, 0, 0.5)` → red with 50% transparency.
    
- `hsl(120, 100%, 50%)` → green.
    
- `hsla(120, 100%, 50%, 0.3)` → transparent green.
    
- `color-mix(in srgb, red 40%, blue)` → blends colors.
    

#### 2. **Math functions**

- `calc(100% - 50px)` → do calculations inside CSS.
    
- `min(50vw, 600px)` → picks the smaller value.
    
- `max(300px, 10%)` → picks the larger value.
    
- `clamp(200px, 50%, 800px)` → restricts a value to a min/max range.
    

#### 3. **Transform functions**

Used with `transform` property:

- `translate(50px, 100px)`
    
- `rotate(45deg)`
    
- `scale(1.5)`
    
- `skew(20deg)`
    
- `matrix(...)` (lower-level version of transform).
    

#### 4. **Shape & geometry functions**

- `circle(50% at center)` (for `clip-path`).
    
- `ellipse(100px 50px at 50% 50%)`
    
- `polygon(0 0, 100% 0, 100% 100%, 0 100%)`
    

#### 5. **Filter functions**

Used with `filter`:

- `blur(5px)`
    
- `brightness(150%)`
    
- `contrast(120%)`
    
- `grayscale(50%)`
    
- `drop-shadow(5px 5px 10px black)`
    

#### 6. **String & attribute functions**

- `attr(data-label)` → gets attribute value (limited use in CSS).
    
- `url("image.png")` → references a file (image, font, etc.).
    

#### 7. **Other useful ones**

- `var(--my-color)` → use CSS custom properties.
    
- `env(safe-area-inset-top)` → environment variables (for iPhone notch, etc.).
    
- `counter(name)` → for CSS counters.
    

----

### Reference 
https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Values_and_Units/CSS_Value_Functions