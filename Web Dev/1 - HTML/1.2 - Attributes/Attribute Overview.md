---
tags:
  - html
  - term
  - fundamental
  - summary
---

- HTML tags often come with **attributes** that provide additional information or functionality. HTML attributes are very similar to these object methods. Each HTML tag (`p`, `div`, `img`, etc.) has a set of element attributes that can be used on them to modify their behavior and styling. There are also a set of global attributes that can be used on ALL HTML tags.

---

### 📌 **Global Attributes** (Applicable to Most HTML Tags)

- **`id`**: Assigns a unique identifier to an element.
    
    ```html
    <p id="intro">Welcome to my website.</p>
    ```
    
- **`class`**: Assigns a class name (for CSS styling or JavaScript targeting).
    
    ```html
    <p class="highlight">This is important!</p>
    ```
    
- **`style`**: Adds inline CSS styles.
    
    ```html
    <h1 style="color: blue;">Hello World</h1>
    ```
    
- **`title`**: Provides additional information (displays as a tooltip).
    
    ```html
    <abbr title="HyperText Markup Language">HTML</abbr>
    ```
    
- **`data-*`**: Custom data attributes for JavaScript.
    
    ```html
    <button data-user-id="123">Click Me</button>
    ```
    
- **`hidden`**: Hides an element.
    
    ```html
    <p hidden>This is hidden text.</p>
    ```
    
- **`tabindex`**: Controls the tab navigation order.
    
    ```html
    <input tabindex="1" type="text" />
    ```
    
- **`lang`**: Specifies the language of content.
    
    ```html
    <p lang="en">Hello!</p>
    ```
    

---

### 🔗 **Attributes for Anchor (`<a>`)**

- **`href`**: Specifies the link URL.
    
    ```html
    <a href="https://example.com">Visit Example</a>
    ```
    
- **`target`**: Opens the link in a new tab or window.
    
    ```html
    <a href="page.html" target="_blank">Open in New Tab</a>
    ```
    
- **`rel`**: Defines the relationship between the linked document and the current one.
    
    ```html
    <a href="page.html" rel="nofollow">No Follow Link</a>
    ```
    

---

### 🖼️ **Attributes for Images (`<img>`)**

- **`src`**: Image source URL.
    
    ```html
    <img src="image.jpg" alt="A beautiful sunset" />
    ```
    
- **`alt`**: Alternative text for accessibility and SEO.
    
    ```html
    <img src="logo.png" alt="Company Logo" />
    ```
    
- **`width`** and **`height`**: Set image dimensions.
    
    ```html
    <img src="image.jpg" width="300" height="200" />
    ```
    

---

### 📋 **Form and Input Attributes**

- **`type`**: Input type (e.g., `text`, `password`, `email`).
    
    ```html
    <input type="text" />
    ```
    
- **`name`**: Input name (used when submitting forms).
    
    ```html
    <input type="text" name="username" />
    ```
    
- **`value`**: Pre-filled value.
    
    ```html
    <input type="text" value="John Doe" />
    ```
    
- **`placeholder`**: Display placeholder text.
    
    ```html
    <input type="text" placeholder="Enter your name" />
    ```
    
- **`required`**: Makes the field mandatory.
    
    ```html
    <input type="email" required />
    ```
    
- **`readonly`**: Makes the field non-editable.
    
    ```html
    <input type="text" value="Locked" readonly />
    ```
    
- **`disabled`**: Disables the input.
    
    ```html
    <input type="text" disabled />
    ```
    
- **`checked`**: Pre-selects a checkbox or radio button.
    
    ```html
    <input type="checkbox" checked />
    ```
    

---

### 📄 **Media-Specific Attributes**

- **`controls`**: Displays media controls.
    
    ```html
    <video controls>
      <source src="video.mp4" type="video/mp4" />
    </video>
    ```
    
- **`autoplay`**: Auto-plays media when the page loads.
    
    ```html
    <audio src="song.mp3" autoplay></audio>
    ```
    
- **`loop`**: Repeats the media.
    
    ```html
    <audio src="song.mp3" loop></audio>
    ```
    

---

### 📊 **Table Attributes**

- **`colspan`**: Merges columns.
    
    ```html
    <td colspan="2">Merged Cell</td>
    ```
    
- **`rowspan`**: Merges rows.
    
    ```html
    <td rowspan="2">Merged Row</td>
    ```
    

---
