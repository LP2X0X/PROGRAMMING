---
tags:
  - js
  - term
  - fundamental
---

In web development, a **CDN (Content Delivery Network)** is a network of servers distributed globally that delivers web content (like JavaScript libraries, images, CSS, fonts, videos, etc.) to users **quickly and efficiently**.

---

## **🔹 Why Use a CDN?**

✅ **Faster load times** (files are served from a server near the user)
✅ **Reduced server load** on your own server
✅ **Better caching** (browsers may already have common CDN files cached)
✅ **High availability** (CDNs have failover and load balancing)
✅ **Easy integration** (just a \<script> or \<link> tag)

---

## **🔹 Example: Using JavaScript Libraries via CDN**
### **📦 jQuery (from jsDelivr)**

```html
<script src="https://cdn.jsdelivr.net/npm/jquery@3.7.1/dist/jquery.min.js"></script>
```

### **🎵 Tone.js (for audio apps)**

```html
<script src="https://cdn.jsdelivr.net/npm/tone@next/build/Tone.js"></script>
```

### **📘 Bootstrap CSS**

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css">
```

---

## **🔹 Popular Free CDNs**

|**CDN**|**Website**|**Notes**|
|---|---|---|
|jsDelivr|https://www.jsdelivr.com/|Supports npm, GitHub, WordPress|
|unpkg|https://unpkg.com/|Delivers npm packages|
|Google CDN|https://developers.google.com/speed/libraries|For popular libraries (e.g., jQuery)|
|Cloudflare CDNJS|https://cdnjs.com/|Large library of open source assets|

---

## **🔹 How to Use a CDN in HTML**

```html
<!DOCTYPE html>
<html>
<head>
  <title>CDN Example</title>
  <script src="https://cdn.jsdelivr.net/npm/lodash@4.17.21/lodash.min.js"></script>
</head>
<body>
  <script>
    console.log(_.shuffle([1, 2, 3, 4])); // lodash function
  </script>
</body>
</html>
```