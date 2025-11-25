---
tags: 
 - react
 - route
 - term
---

With React Router v6.4+, the recommended way to define routes shifts from the JSX `<Routes>` component to the **Data Router** objects using `createBrowserRouter`.

While the component part (using `<Outlet />`) stays the same, the **configuration** changes significantly to unlock powerful features like Loaders, Actions, and Error Boundaries.

### 1. The Configuration: `createBrowserRouter`

Instead of nesting `<Route>` components, you now use an **array of objects**. A Layout Route is simply an object that has both an `element` (your layout) and a `children` property.

JavaScript

```
import { createBrowserRouter, RouterProvider, Outlet } from 'react-router-dom';

// 1. Define your Layout Component (Same as before)
const AppLayout = () => {
  return (
    <div>
      <nav>Navbar</nav>
      <div className="content">
        {/* Child routes render here */}
        <Outlet />
      </div>
      <footer>Footer</footer>
    </div>
  );
};

// 2. Define the Router using the Array syntax
const router = createBrowserRouter([
  {
    path: "/",
    element: <AppLayout />, // The Parent (Layout)
    children: [
      {
        index: true, // Matches path: "/"
        element: <Home />,
      },
      {
        path: "about",
        element: <About />,
      },
      {
        path: "dashboard",
        element: <Dashboard />,
        // Layouts can also have their own Loaders in v6.4+
        loader: dashboardLoader, 
      },
    ],
  },
]);

// 3. Render the Provider
function App() {
  return <RouterProvider router={router} />;
}
```

---

### 2. The Alternative: JSX with `createRoutesFromElements`

If you prefer the visual style of JSX but still want the v6.4+ Data APIs (loaders/actions), you can use the `createRoutesFromElements` helper. This is functionally identical to the array method above.

JavaScript

```
import { 
  createBrowserRouter, 
  createRoutesFromElements, 
  Route, 
  RouterProvider 
} from 'react-router-dom';

const router = createBrowserRouter(
  createRoutesFromElements(
    <Route path="/" element={<AppLayout />}>
      <Route index element={<Home />} />
      <Route path="about" element={<About />} />
    </Route>
  )
);
```

---

### 3. Key v6.4+ Features in Layouts

The move to `createBrowserRouter` makes Layout Routes significantly more powerful. It's no longer just about shared UI; it's about shared **lifecycle and data**.

#### A. Error Boundaries (`errorElement`)

In v6.4, you can define an `errorElement` on your Layout Route. If _any_ child route crashes or throws a 404, the error bubbles up to the Layout. The Layout can then swap the `<Outlet />` for your Error Element while keeping the Navbar and Footer visible.

JavaScript

```
{
  path: "/",
  element: <RootLayout />,
  errorElement: <ErrorPage />, // Catches errors from Home, About, etc.
  children: [ ... ]
}
```

#### B. Layout Loaders

You can attach a `loader` to a Layout Route. This is perfect for fetching data that every page needs, such as:

- User authentication status
    
- Shopping cart count for the Navbar
    
- Global theme settings
    

When you navigate to a child, the Parent Layout's loader fetches _in parallel_ with the child's loader.

#### C. Scroll Restoration

When using the Data Router, adding `<ScrollRestoration />` to your root layout automatically manages scroll positions, ensuring users return to the same scroll point when hitting the "Back" button.

JavaScript

```
import { ScrollRestoration, Outlet } from 'react-router-dom';

function RootLayout() {
  return (
    <div>
      <Nav />
      <Outlet />
      <ScrollRestoration /> {/* Handles scroll position automatically */}
    </div>
  );
}
```

### Summary: v6.0 vs v6.4+

|**Feature**|**v6.0 (Standard)**|**v6.4+ (Data Router)**|
|---|---|---|
|**Definition**|`<Routes>` component|`createBrowserRouter` objects|
|**Nesting**|JSX nesting|`children` array property|
|**Data Fetching**|`useEffect` inside components|`loader` function in route config|
|**Error Handling**|Manual boundaries|`errorElement` property|

**Would you like to see how to implement a `loader` inside a Layout Route to fetch current user data for a Navbar?**