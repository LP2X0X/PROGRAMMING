---
tags: 
 - react
 - router
 - strategy
---

**Definition**: In this strategy, the component is rendered first, and the data fetching happens after the component has mounted. Typically, this is done using the useEffect hook in React.

#### How it Works:

- The component renders, showing a loading state (e.g., spinner or placeholder).

- The useEffect hook triggers the data fetch.

- After the data is fetched, the component updates with the new data.

#### Example:

```jsx

import { useEffect, useState } from "react";

function UserProfile() {

  const [data, setData] = useState(null);

  const [loading, setLoading] = useState(true);

  useEffect(() => {

    async function fetchData() {

      const response = await fetch("/api/user");

      const result = await response.json();

      setData(result);

      setLoading(false);

    }

    fetchData();

  }, []);

  if (loading) return <p>Loading...</p>;

  return <div>{data.name}</div>;

}        

```

#### Pros:

- Simple to implement, especially for smaller apps.

- Standard React pattern using useEffect.

#### Cons:

- Slower page loads: The component renders before data is ready, meaning users see a loading state.

- Data loading waterfalls: If multiple components depend on data fetching, each one may trigger a new round of requests, causing delays and unnecessary re-renders.

---

### When to Use 

- Works well for small apps or non-critical data.

- Suitable when immediate data is not essential for the initial load (e.g., fetching after user interaction).