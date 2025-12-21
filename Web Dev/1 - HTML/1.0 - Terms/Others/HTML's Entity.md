---
tags:
  - css
  - term
  - fundamental
---

An **HTML entity** is a special code used to display reserved characters, symbols, or characters that are difficult to type directly in HTML. They start with `&` and end with `;`.

---

There are **three types of HTML entities**, categorized by **how they are written**.

## 1. **Named entities**

Named references start with an ampersand sign (`&`) and end with a semicolon (`;`). By using a named character reference, the HTML parser will not confuse this with an actual HTML element.
Use a **human-readable name**.

```html
&nbsp;   <!-- non-breaking space -->
&lt;     <!-- < -->
&gt;     <!-- > -->
&amp;    <!-- & -->
```

✔ Easy to read  
✘ Limited set

### 2. **Decimal numeric entities**

Decimal numeric reference starts with an ampersand sign and hash symbol (`#`), followed by one or more decimal digits, followed by a semicolon.
Use a **base-10 number** (Unicode code point).

```html
&#160;   <!-- non-breaking space -->
&#60;    <!-- < -->
&#38;    <!-- & -->
```

✔ Covers all Unicode characters  
✔ Works everywhere  
✘ Harder to read

### 3. **Hexadecimal numeric entities**

Hexadecimal numeric reference starts with an ampersand sign, hash symbol, and the letter `x`. Then it is followed by one or more ASCII hex digits and ends with a semicolon.
Use a **base-16 number**.

```html
&#xA0;   <!-- non-breaking space -->
&#x3C;   <!-- < -->
&#x26;   <!-- & -->
```

✔ Covers all Unicode characters  
✔ Compact for Unicode tables  
✘ Least readable

---

## Rule of thumb

- Use **named entities** for common symbols
    
- Use **numeric entities** when no named entity exists
    

That’s the full picture, no extras.

### **Common HTML Entities**
| Character | Entity Code | Description   |
| --------- | ----------- | ------------- |
| `<`       | `&lt;`      | Less than     |
| `>`       | `&gt;`      | Greater than  |
| `&`       | `&amp;`     | Ampersand     |
| `"`       | `&quot;`    | Double quote  |
| `'`       | `&apos;`    | Apostrophe    |
| `©`       | `&copy;`    | Copyright     |
| `®`       | `&reg;`     | Registered    |
| `™`       | `&trade;`   | Trademark     |
| `€`       | `&euro;`    | Euro          |
| `£`       | `&pound;`   | British Pound |
| `¥`       | `&yen;`     | Yen           |

### **Example in HTML**
```html
<p>5 &lt; 10 is true.</p>
<p>Copyright &copy; 2025</p>
```
This will render as:
```
5 < 10 is true.
Copyright © 2025
```