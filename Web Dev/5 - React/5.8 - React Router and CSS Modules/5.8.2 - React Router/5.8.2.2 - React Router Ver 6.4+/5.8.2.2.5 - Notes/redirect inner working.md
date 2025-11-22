---
tags: 
 - react
 - router 
 - redirect
 - note
---

# 🎯 **What actually happens when you call `redirect()` in React Router?**

When you call:

```js
return redirect("/login");
```

You are **not** returning JSX.  
You are **not** navigating manually.

Instead, React Router creates a **special Response object** that looks like a real HTTP redirect.

---

# 🧩 **What does this Response look like?**

`redirect("/login")` creates something like this:

```js
new Response(null, {
  status: 302,
  headers: {
    Location: "/login"
  }
});
```

This is the same format as what you’d get from a server redirect.

React Router detects this Response inside the loader/action and says:

> “Oh, this is a redirect response.  
> Instead of rendering the route, I will navigate the browser to the Location header.”

So React Router performs a **client-side navigation** to `/login`.

No rendering happens.  
No error boundary triggers.  
No component runs.

The router handles the redirect _before_ anything is drawn.

---

# 🔥 **Why does React Router use a Response object?**

Because React Router 6.4+ is designed to work like the Web Fetch API.

Loaders and actions behave like:

- async functions
    
- that can return responses
    
- similar to real server endpoints
    

This makes the router feel like a **mini backend** inside your frontend.

By returning a redirect Response, it becomes easy to:

- mirror real server logic
    
- reuse the same mental model
    
- support server-side rendering (SSR) with the same API
    
- support full-stack frameworks like Remix
    

---

# 🏗 **Flow illustration**

### ❌ Old approach (React Router before 6.4)

Loader returns data → Component decides → useNavigate()

```
loader → component → useEffect → navigate()
```

This causes:

- extra renders
    
- flicker jumps
    
- complex logic
    

---

### ✅ New approach (6.4+)

Loader returns redirect Response → Router redirects immediately

```
loader → redirect Response → router navigates → new route loads
```

No rendering of the original route.

---

# 🧪 Concrete example

```js
export async function loader() {
  const user = await getUser();
  if (!user) {
    return redirect("/login");
  }
  return user;
}
```

### What happens behind the scenes:

1. `loader()` runs
    
2. detect no user
    
3. `redirect("/login")` creates a Response object
    
4. React Router sees status `302`
    
5. React Router **never loads the protected route**
    
6. React Router redirects to `/login`
    
7. `/login` loader runs
    
8. Login page renders normally
    

No flicker.  
No component load.  
No weird timing issues.

---

# 📦 Summary (simple version)

> **`redirect()` returns a fake HTTP redirect Response.  
> React Router recognizes this as a redirect instruction  
> and navigates to the new URL immediately on the client.**

This gives:

- instant navigations
    
- no rendering of unwanted pages
    
- server-like correctness
    
- SSR compatibility
    
- cleaner logic in loaders & actions
    
