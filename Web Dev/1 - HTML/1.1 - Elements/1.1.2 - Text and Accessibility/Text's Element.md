---
tags:
  - html
  - term
  - fundamental
---

**0. `<strong>` (Strong Importance)**

• Renders text in **bold by default**, **with semantic importance**.

• Indicates that the content is **important, serious, or urgent**.

• Screen readers typically announce it with **strong emphasis**, improving accessibility.

**Example:**

```html
<p>This is a <strong>very important</strong> message.</p>
```

---

**1. \<b> (Bold)**

• Renders text in **bold**, but **without semantic importance**.

• Use it for styling, not for emphasizing meaning.

**Example:**

```html
<p>This is a <b>bold</b> word.</p>
```

---

**2. \<em> (Emphasis - Italics)**

• Used to **emphasize** text, typically displayed in _italics_.

• Often read with **stronger intonation** by screen readers.
  
**Example:**

```html
<p>This is an <em>important</em> word.</p>
```

---

**3. \<i> (Italic)**

• Renders text in _italics_ but **without emphasis or importance**.

• Often used for terms, technical jargon, or foreign words.

**Example:**

```html
<p>The word <i>schadenfreude</i> is German.</p>
```

---

**4. \<mark> (Highlighted Text)**

• Highlights text, typically with a **yellow background**.

• Used to indicate relevance in search results or highlight key points.

**Example:**

```html
<p>Please read the <mark>important</mark> details.</p>
```

---

**5. \<u> (Underlined)**

• Underlines text **without additional meaning**.

• Previously used for links but now mostly used for annotations.

**Example:**

```html
<p>This is an <u>underlined</u> word.</p>
```

---

**6. \<small> (Smaller Text)**

• Displays text in a **smaller font size**.

• Often used for disclaimers, copyright, or fine print.

**Example:**

```html
<p><small>Terms and conditions apply.</small></p>
```

---

**7. \<del> (Deleted Text)**

• Shows **strikethrough** text, indicating **removal or edits**.

**Example:**

```html
<p>The price was <del>$50</del> now it's $30.</p>
```

---

**8. \<ins> (Inserted Text)**

• Underlines text to indicate an **addition** or change.

**Example:**

```html
<p>The correct price is <ins>$30</ins>.</p>
```

---

**9. \<sup> (Superscript) & \<sub> (Subscript)**

• \<sup> raises text (e.g., exponents, footnotes).

• \<sub> lowers text (e.g., chemical formulas).

**Example:**

```html
<p>Water formula: H<sub>2</sub>O</p>
<p>Square: x<sup>2</sup></p>
```

---


**10. \<span> **

• The most generic text's element. It doesn't mean anything on its own, but can be used together with the global attributes, e.g. class, lang,...

**Example:**

```html
<p>This is a <span style="color: red;">highlighted</span> word.</p>
```

---
## **Summary Table**

| **Element** | **Default Style** | **Meaning**                         |
| ----------- | ----------------- | ----------------------------------- |
| \<strong>   | Bold              | Important text                      |
| \<b>        | Bold              | Visual emphasis (no meaning)        |
| \<em>       | Italic            | Emphasized text                     |
| \<i>        | Italic            | Foreign words, terms, or thoughts   |
| \<mark>     | Highlight         | Highlighted text                    |
| \<u>        | Underline         | Annotated text (not for links)      |
| \<small>    | Smaller text      | Legal notes, fine print             |
| \<del>      | Strikethrough     | Deleted text                        |
| \<ins>      | Underlined        | Inserted text                       |
| \<sup>      | Raised            | Superscript (e.g., exponents)       |
| \<sub>      | Lowered           | Subscript (e.g., chemical formulas) |
| \<span>     | Generic           | Generic style                       |
