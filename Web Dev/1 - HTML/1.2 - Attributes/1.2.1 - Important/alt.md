---
tags: 
 - html
 - attribute
 - important
---

# 🔶 **What is `alt`?**

`alt` stands for **alternative text**.  
It provides a **text description of an image** when the image **cannot be seen**.

Used only on:

```html
<img src="..." alt="...">
```

---

# 🔶 **Why `alt` exists**

It serves 3 purposes:

### 1️⃣ **Accessibility** (screen readers)

People who cannot see the image hear the `alt` text.

### 2️⃣ **Fallback**

If the image fails to load, the `alt` text appears instead.

### 3️⃣ **SEO**

Search engines use alt text to understand images.

---

# 🔶 **How to write good `alt` text**

### ✔ Describe the image’s _content or purpose_

```html
<img src="dog.jpg" alt="Golden retriever running in a field">
```

### ✔ If the image is decorative → empty alt

This hides it from screen readers.

```html
<img src="divider.png" alt="">
```

### ✔ If the image is a button → describe the action

```html
<button>
  <img src="save.png" alt="Save">
</button>
```

### ✔ If the image contains text → include the text

```html
<img src="banner.png" alt="Special sale: 50% off">
```

---

# 🔶 **When NOT to use alt text**

Never repeat words like “image of” or “picture of”.

❌ Bad:

```html
<img src="cat.jpg" alt="Image of a cat">
```

✔ Good:

```html
<img src="cat.jpg" alt="A cat sleeping on a sofa">
```

---

# 🔶 **Empty `alt` (`alt=""`) - important rule**

You should use `alt=""` when the image is **purely decorative**, like:

- background textures
    
- UI icons that convey no meaning
    
- spacing/padding images (rare today)
    

Screen readers skip it completely.

---

# 🔶 **Missing `alt` vs empty `alt`**

|Case|Screen reader behavior|Meaning|
|---|---|---|
|`alt="text"`|Reads “text”|Image has meaningful content|
|`alt=""`|Skips it|Decorative image|
|**No `alt` attribute**|Reads the file name|BAD accessibility|

Never omit the `alt` attribute.

---

# 🔶 **Examples**

### 1. Meaningful image

```html
<img src="/avatars/john.png" alt="John Doe's profile picture">
```

### 2. Decorative

```html
<img src="line.svg" alt="">
```

### 3. Product picture

```html
<img src="shoe.jpg" alt="Red running shoes with white sole">
```

### 4. Company logo (important)

```html
<img src="logo.svg" alt="OpenAI logo">
```

---

# 🔶 Summary (Cheat Sheet)

- `alt` describes an image's **content or purpose**
    
- Required for accessibility
    
- Use `alt=""` for **decorative** images
    
- Never omit the attribute
    
- Don’t say “image of”
    
- Helps SEO and fallback text
    