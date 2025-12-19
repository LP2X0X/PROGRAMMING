---
tags:
  - css
  - term
  - fundamental
---

An **HTML entity** is a special code used to display reserved characters, symbols, or characters that are difficult to type directly in HTML. They start with `&` and end with `;`.

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