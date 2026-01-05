---
tags: 
 - html
 - attribute
---

The `loading` attribute is a powerful performance optimization tool used to control **when** the browser should start downloading an iframe or image. It is a key part of modern web vitals (speed) scores.

---

## 1. The Core Values

There are two primary values you can assign to the `loading` attribute:

| **Value**  | **Description**                                                                               | **Best Use Case**                                   |
| ---------- | --------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| **`lazy`** | Defers loading until the element reaches a calculated distance from the viewport (scrolling). | Content "below the fold" (not visible immediately). |
|**`eager`**|Loads the resource immediately, regardless of where it is on the page.|Hero images or primary content at the top of the page.|

---

## 2. Why Use `loading="lazy"`?

By default, browsers try to download everything in the HTML as fast as possible. If you have 5 YouTube iframes at the bottom of a long article, the browser will download them all at once, slowing down the initial page load for the user.

- **Saves Data:** Users who never scroll to the bottom won't waste data downloading content they never see.
    
- **Prioritizes Resources:** It allows the browser to focus its "bandwidth" on the text and images the user is actually looking at right now.
    
- **Better SEO:** Page speed is a ranking factor; lazy loading is one of the easiest ways to improve your **Largest Contentful Paint (LCP)**.
    

---

## 3. Implementation Example

For iframes, the implementation is simple:

```html
<iframe 
  src="https://www.google.com/maps/embed..." 
  loading="lazy"
  title="Our Office Location">
</iframe>
```

---

## 4. Best Practices & Rules

### A. Don't Lazy Load "Above the Fold"

Never use `loading="lazy"` for elements at the very top of your page (like your logo or a header video). This will actually **delay** their appearance and hurt your user experience. For those, use `loading="eager"`.

### B. Use Width and Height

When using `loading="lazy"`, always specify `width` and `height` attributes. This prevents "layout shift" where the page jumps around as the iframe finally loads and takes up space.

### C. The Browser Decides the Distance

You cannot control exactly how many pixels away the user must be before the loading starts. Each browser (Chrome, Firefox, Safari) has its own internal logic based on the user's connection speed. If the user is on a slow 3G connection, the browser will start "lazy loading" much earlier to ensure the content is ready by the time they scroll to it.

---

## 5. Browser Support

The `loading` attribute is supported in all modern browsers. For very old browsers, the attribute is simply ignored, and the iframe/image will load immediately (the default behavior).

| **Feature**             | **Chrome** | **Firefox** | **Safari** | **Edge**  |
| ----------------------- | ---------- | ----------- | ---------- | --------- |
| **Iframe Lazy Loading** | Supported  | Supported   | Supported  | Supported |