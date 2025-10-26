---
tags: 
 - js
 - url
 - encoding
---

**1. What is URL encoding?**

- URLs can only contain certain characters (ASCII letters, digits, and `- _ . ~`).
    
- Other characters (like spaces, Unicode, or reserved symbols) must be _encoded_ as `%` followed by their hexadecimal code (e.g. space → `%20`).
    

```javascript
// using some cyrillic characters for this example

let url = new URL('https://ru.wikipedia.org/wiki/Тест');

url.searchParams.set('key', 'ъ');
alert(url); //https://ru.wikipedia.org/wiki/%D0%A2%D0%B5%D1%81%D1%82?key=%D1%8A
```

**2. Encoding in practice:**

- JavaScript provides built-in functions for encoding/decoding parts of URLs:
    
    - `encodeURI(url)` – encodes an entire URL (but keeps special characters like `:`, `/`, `?`, `#`).
        
    - `decodeURI(url)` – decodes what `encodeURI` encodes.
        
    - `encodeURIComponent(str)` – encodes a component (like query parameters), encoding _everything except_ letters, digits, and `- _ . ~`.
        
    - `decodeURIComponent(str)` – decodes that.
        

**3. When to use what:**

- Use `encodeURI()` for the **whole URL**.
    
- Use `encodeURIComponent()` for **query strings or path parameters** (parts of the URL).
    

**Example:**

```js
// using cyrillic characters in url path
let url = encodeURI('http://site.com/привет');

alert(url); // http://site.com/%D0%BF%D1%80%D0%B8%D0%B2%D0%B5%D1%82
```

```js
let param = "John Doe & Co";
let url = "https://example.com/?name=" + encodeURIComponent(param);
// https://example.com/?name=John%20Doe%20%26%20Co
```

**4. Automatic Encoding:**

- When using the `URL` or `URLSearchParams` APIs in JS, encoding is handled automatically.
    