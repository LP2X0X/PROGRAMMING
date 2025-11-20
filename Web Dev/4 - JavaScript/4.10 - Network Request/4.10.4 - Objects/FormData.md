---
tags: 
 - js
 - network
 - object
---

The **`FormData` interface** is a native web API object designed to easily construct a set of key/value pairs representing form fields and their values. It simplifies the process of sending form data via methods like `XMLHttpRequest` or the modern `fetch()` API, especially when dealing with file uploads.

React Router's `<Form>` component uses the `FormData` object internally to package the data it sends to your route's `action` function.

---

## 📝 Key Details and Usage

### 1. What is `FormData`?

- It's a way to programmatically build an object that mirrors the data structure a standard HTML form would submit to a server.
    
- It's an iterable object, meaning you can loop through its key/value pairs using `for...of` or methods like `.entries()`.
    
- It is the standard mechanism for handling multipart/form-data requests, which are essential for **file uploads**.
    

### 2. Creation and Population

You can create a `FormData` object in two main ways:

#### A. From an HTML Form Element (Most Common)

Passing an existing `<form>` element to the constructor automatically populates the `FormData` object with all of the form's fields and their current values (based on their `name` attributes).

JavaScript

```
const myForm = document.getElementById('user-form');
const formData = new FormData(myForm); 
// formData now contains all named inputs from myForm
```

#### B. Programmatic Creation

You can create an empty object and manually append key/value pairs.

JavaScript

```
const formData = new FormData();
formData.append('username', 'Alice');
formData.append('email', 'alice@example.com');

// For files, the value is typically a File or Blob object
const blob = new Blob(['Hello, world!'], { type: 'text/plain' });
formData.append('file', blob, 'greeting.txt'); 
```

### 3. Core Methods

The `FormData` object provides several methods for interacting with its stored data:

|**Method**|**Description**|
|---|---|
|**`.append(name, value, filename)`**|Adds a new key/value pair. If the key already exists, the new value is **appended**, creating multiple entries for that key (useful for multi-selects).|
|**`.set(name, value, filename)`**|Sets a new key/value pair. If the key already exists, the old value(s) are **overwritten**.|
|**`.get(name)`**|Returns the value of the **first** entry with the given key.|
|**`.getAll(name)`**|Returns an **array** of all values associated with the given key.|
|**`.delete(name)`**|Removes all entries associated with the given key.|
|**`.has(name)`**|Returns a boolean indicating whether the object contains an entry for the given key.|

### 4. Integration with React Router `action`

In React Router, the `action` function receives a standard **`request`** object, and you extract the `FormData` from it:

JavaScript

```
// In your route action function:
export async function newPostAction({ request }) {
  // 1. Get the FormData object from the request
  const formData = await request.formData(); 

  // 2. Extract values using .get()
  const title = formData.get('title');
  const content = formData.get('content');
  
  // 3. Optional: Convert to a plain JavaScript object
  const postData = Object.fromEntries(formData); 
  
  // 4. Perform your API call/mutation
  const response = await fetch('/api/posts', {
    method: 'POST',
    body: formData, // You can send the FormData object directly in the fetch body
  });
  
  // ... return redirect or data
}
```

By receiving the `FormData` object directly, React Router ensures that data submitted via the `<Form>` component, including files, is readily available to your data mutation logic.

---

### Other notes
https://stackoverflow.com/questions/70404820/why-does-object-fromentriesformdata-work