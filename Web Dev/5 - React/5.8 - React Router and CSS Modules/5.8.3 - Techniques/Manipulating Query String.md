---
tags: 
 - react
 - router
 - technique
---

## 🧩 1. Basic setup

Import the hook:

```js
import { useSearchParams } from "react-router-dom";
```

Then inside your component:

```jsx
const [searchParams, setSearchParams] = useSearchParams();
```

✅ `searchParams` → lets you _read_ query values.  
✅ `setSearchParams` → lets you _update_ the query string in the URL.

---

## 🧭 2. Reading query parameters

```jsx
const sort = searchParams.get("sort");
const filter = searchParams.get("filter");

console.log(sort, filter);
```

Example URL:

```
/cities?sort=asc&filter=europe
```

→ `sort = "asc"`, `filter = "europe"`

---

## 🧩 3. Updating (adding or changing) query params

You can replace the whole set:

```jsx
setSearchParams({ sort: "desc", filter: "asia" });
```

✅ URL becomes:

```
/cities?sort=desc&filter=asia
```

---

## ⚙️ 4. Updating only one key while keeping others

Since `setSearchParams` **overwrites** everything, you usually do:

```jsx
function handleSortChange() {
  const params = new URLSearchParams(searchParams);
  params.set("sort", "desc"); // update one
  setSearchParams(params);    // reapply
}
```

✅ Keeps existing params, just changes one.

---

## ❌ 5. Deleting a query param

```jsx
const params = new URLSearchParams(searchParams);
params.delete("filter");
setSearchParams(params);
```

✅ Removes `filter` from the URL.

---

## 🧠 6. Full Example

```jsx
import { useSearchParams } from "react-router-dom";

function CityList() {
  const [searchParams, setSearchParams] = useSearchParams();

  const handleSort = () => {
    const params = new URLSearchParams(searchParams);
    const currentSort = params.get("sort");
    params.set("sort", currentSort === "asc" ? "desc" : "asc");
    setSearchParams(params);
  };

  return (
    <div>
      <button onClick={handleSort}>Toggle Sort</button>
      <p>Current sort: {searchParams.get("sort") || "none"}</p>
    </div>
  );
}
```

✅ Clicking the button toggles:

```
/cities?sort=asc
/cities?sort=desc
```

---

### 🧠 TL;DR

|Action|Code|Effect|
|---|---|---|
|Read|`searchParams.get("key")`|Get value|
|Add/update|`params.set("key", "value")` + `setSearchParams(params)`|Update one param|
|Delete|`params.delete("key")`|Remove from URL|