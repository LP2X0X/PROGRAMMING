---
tags: 
 - react
 - router
 - technique
---

## 🔍 Reading Query Strings

React Router doesn’t automatically parse query strings for you — but you can access them easily using the `useLocation` hook.

### Example:

```jsx
import { useLocation } from "react-router-dom";

function SearchPage() {
  const { search } = useLocation(); // returns the query string like "?q=apple&page=2"
  const queryParams = new URLSearchParams(search);

  const query = queryParams.get("q");     // "apple"
  const page = queryParams.get("page");   // "2"

  return (
    <div>
      <h2>Search: {query}</h2>
      <p>Page: {page}</p>
    </div>
  );
}

export default SearchPage;
```

If you go to `/search?q=apple&page=2`, it’ll render:

```
Search: apple
Page: 2
```

---

## ✏️ Updating Query Strings

To change the query string from code, use the `useNavigate` hook:

```jsx
import { useNavigate } from "react-router-dom";

function FilterButton() {
  const navigate = useNavigate();

  function applyFilter() {
    navigate("/search?q=banana&page=1");
  }

  return <button onClick={applyFilter}>Apply Filter</button>;
}

export default FilterButton;
```

---

## 🧰 Bonus: useSearchParams (React Router v6.4+)

A cleaner way to manage query strings.

```jsx
import { useSearchParams } from "react-router-dom";

function SearchPage() {
  const [searchParams, setSearchParams] = useSearchParams();

  const query = searchParams.get("q") || "";
  const page = searchParams.get("page") || "1";

  function nextPage() {
    setSearchParams({ q: query, page: Number(page) + 1 });
  }

  return (
    <div>
      <h2>Query: {query}</h2>
      <p>Page: {page}</p>
      <button onClick={nextPage}>Next Page</button>
    </div>
  );
}

export default SearchPage;
```

✅ This approach keeps the URL and component state automatically in sync.