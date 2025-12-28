---
tags: 
 - react
 - router
 - loader
 - hook
 - custom
---

The `useLoaderData` hook in React Router is the primary way to access the data returned by a route's **`loader` function** within the route's component.

---

## 💡 Compact Overview

|**Feature**|**Description**|
|---|---|
|**Purpose**|To read the data fetched by the route's associated `loader` function.|
|**Timing**|The hook returns the data **only after** the `loader` has successfully completed.|
|**Location**|Must be called inside the **component** that is rendered by the route where the `loader` is defined, or one of its descendants.|
|**Return Value**|The exact value (object, array, string, etc.) that the `loader` function resolved or returned.|
|**State Management**|Eliminates the need for manual `useState` and `useEffect` for the initial data fetch.|

---

## 📝 Example

### 1. The Loader (Data Source)

```js
// A simple loader fetching a post by ID
export async function postLoader({ params }) {
  const response = await fetch(`/api/posts/${params.postId}`);
  return response.json(); // Returns the post object
}
```

### 2. The Component (Data Access)

```js
import { useLoaderData } from 'react-router-dom';

function PostDetail() {
  // 'post' automatically gets the object returned by postLoader
  const post = useLoaderData(); 
  
  return (
    <article>
      <h2>{post.title}</h2>
      <p>{post.body}</p>
    </article>
  );
}

export default PostDetail;
```