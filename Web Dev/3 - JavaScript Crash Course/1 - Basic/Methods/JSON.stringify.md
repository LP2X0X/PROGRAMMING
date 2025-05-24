---
tags: js, method
---

- The JSON.stringify method converts a JavaScript object into a JSON string. 

```js
JSON.stringify(nested);
'{"name":"Outer","content":{"name":"Middle","content":{"name":"Inner","content":"Whoa…"}}}
```

- All that’s missing from this representation are the original line breaks and indentations we used to clarify the object literal’s nested structure. To re-create those, we can pass JSON.stringify another argument representing the number of spaces to indent each new nested object:

```js
JSON.stringify(value, replacer, space)
```

|**Parameter**|**Description**|
|---|---|
|value|Object or value to stringify|
|replacer (optional)|Function or array to filter properties|
|space    (optional)|Number or string for pretty printing|