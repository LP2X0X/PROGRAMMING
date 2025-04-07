The object-fit CSS property is used to control **how media elements** like images or videos are **resized to fit their container**.

  

It’s especially useful when you’re working with responsive designs or fixed-size containers and want to make sure the image/video looks good without distortion.

---

**📌 Syntax:**

```css
object-fit: fill | contain | cover | none | scale-down;
```

Apply it like this:

```css
<img src="image.jpg" style="width: 300px; height: 200px; object-fit: cover;">
```

  

---

**🔑 Values:**

|**Value**|**Description**|
|---|---|
|fill|**Default**. The image stretches to fill the container, possibly distorting it.|
|contain|The entire image is visible inside the container — no cropping, but there may be empty space (letterboxing).|
|cover|The image fills the container completely — might crop parts of it, but keeps aspect ratio.|
|none|No resizing — image keeps original size.|
|scale-down|The image is either none or contain, whichever is smaller.|

  

---

**✅ Example:**

```html
<img src="cat.jpg" style="width: 300px; height: 200px; object-fit: contain;">
<img src="dog.jpg" style="width: 300px; height: 200px; object-fit: cover;">
```

  

---

**💡 Bonus Tips:**

• Works well with fixed width and height containers.

• Use alongside object-position to control **alignment** (like center, top, bottom right, etc.).
