---
tags:
  - html
  - term
  - fundamental
---

### **🌐 What is XHTML ?**


**XHTML** stands for **Extensible HyperText Markup Language**.

It is a **stricter**, **XML-compliant** version of HTML.

---

### **✅ Key Concepts**

|**Feature**|**HTML**|**XHTML**|
|---|---|---|
|Syntax lenient|Yes|No|
|Based on|SGML|XML|
|Tags closing|Optional sometimes|**Required**|
|Case sensitivity|Not case-sensitive|**Tag names must be lowercase**|
|Quotes|Optional for attributes|**Required** for all attributes|
|Doctype|Loose/strict/HTML5|Specific XHTML doctype required|
|Parsing|Browser HTML parser|**XML parser**|

---

### **📄 Example: HTML vs XHTML**

  

#### **✅ HTML**

```
<img src="cat.jpg">
```

#### **✅ XHTML**

```
<img src="cat.jpg" />
```

---

### **🔧 XHTML Rules Summary**

1. **All tags must be closed**
    

```
<br />  <img src="..." />  <hr />
```

2. **Attribute values must be quoted**
    

```
<input type="text" value="hello" />
```

3. **Tags must be lowercase**
    

```
<html> not <HTML>
```

4. **Must have one root element**
    

```
<html> ... </html>
```

5.**Must be well-formed XML**
    

---

### **📑 XHTML Doctype Example**

```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
   "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
```

---

### **🆚 When to Use XHTML**

|**Use XHTML if…**|
|---|
|You need strict validation|
|You’re working with XML-based tools|
|You want consistent behavior across platforms|

But nowadays, **HTML5** is preferred unless XHTML is specifically required.
