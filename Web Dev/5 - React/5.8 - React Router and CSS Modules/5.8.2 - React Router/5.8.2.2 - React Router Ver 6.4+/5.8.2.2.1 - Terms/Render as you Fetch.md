---
tags: 
 - react
 - router
 - strategy
---

### 2️⃣ Render-as-You-Fetch

Definition: In this strategy, data is fetched in parallel with rendering the component, ensuring the data is available before the component is rendered. React Router's loader function is an example of this approach.

#### How it Works:

- The data fetch begins at the same time the component starts rendering.

- Data is fetched as part of the routing process, so when the component is ready to render, the data is already available.

- The component uses the fetched data immediately, without needing an additional loading state.

#### Example:

```jsx

import { useLoaderData } from "react-router-dom";

function UserProfile() {

  const data = useLoaderData(); // Fetch data before rendering

  return <div>{data.name}</div>;

}

// Loader function for data fetching

export async function loader() {

  const response = await fetch("/api/user");

  return await response.json();

}        

```

#### Pros:

- Faster page loads: Data is already fetched when the component renders, reducing loading times.

- Eliminates loading spinners: Since data is pre-fetched, there's no need for a loading state.

- Prevents data loading waterfalls: All necessary data is fetched upfront, avoiding the delays caused by sequential data requests in child components.

#### Cons:

- More complex to implement in some cases, especially if you are transitioning from the traditional useEffect pattern.

- Can be overkill for simple applications with very little data dependency.

### When to Use 

- Ideal for larger apps or critical data where fast page loads are important.

- Prevents multiple re-renders and improves performance by fetching data early.

- Useful when dealing with nested components that rely on data, to avoid waterfalls.