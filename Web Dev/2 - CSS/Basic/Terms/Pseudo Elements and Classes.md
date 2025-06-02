**Pseudo-elements** and **pseudo-classes** in CSS are used to style specific parts of an element or to select elements based on their state. While they may sound similar, they serve distinct purposes:

---

### 🟢 **Pseudo-classes**

- Target elements **based on their state** or **position** in the document.
- Prefixed with a **single colon (`:`)**.

✅ **Examples of common pseudo-classes:**

```css
/* Style when the user hovers over a button */
button:hover {
  background-color: lightblue;
}

/* Style the first child of a parent element */
p:first-child {
  font-weight: bold;
}

/* Style links that have already been visited */
a:visited {
  color: purple;
}
```

📌 **Other useful pseudo-classes:**

- `:hover` – When the user hovers over an element.
- `:focus` – When an element (e.g., input) is focused.
- `:nth-child(n)` – Selects the nth child of a parent.
- `:not()` – Excludes elements that match the selector.

---

### 🔵 **Pseudo-elements**

- Style specific **parts of an element** (like the first letter) or **insert content**.
- Prefixed with **double colons (`::`)** (CSS3 standard).

✅ **Examples of common pseudo-elements:**

```css
/* Style the first letter of a paragraph */
p::first-letter {
  font-size: 2rem;
}

/* Add content before an element */
h1::before {
  content: "🔥 ";
}

/* Style placeholder text in input */
input::placeholder {
  color: gray;
}
```

📌 **Other useful pseudo-elements:**

- `::before` – Inserts content **before** an element.
- `::after` – Inserts content **after** an element.
- `::first-line` – Targets the **first line** of text.
- `::selection` – Styles text selected by the user.

---

### 🔥 **Key Differences**

|Feature|Pseudo-class|Pseudo-element|
|---|---|---|
|**Purpose**|Select elements based on state|Style parts of an element|
|**Syntax**|`:` (single colon)|`::` (double colon)|
|**Examples**|`:hover`, `:focus`|`::before`, `::after`|
|**Usage**|Works on whole elements|Works on sub-parts of an element|
