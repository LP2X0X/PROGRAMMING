---
tags: 
 - react
 - router
 - term
---

![[bad.png|center|700]]

This screenshot is from the **Network tab** in browser devtools.  
Each row is a **network request**, and the green/blue bars show how long each request took and _when it started_.

The important thing:

### 👉 The requests are NOT happening in parallel.

They are happening **one after another** — in a _waterfall_.

---

# 🧠 **Why does this happen? (The real reason)**

### Because the code that requests the data **is itself lazy-loaded**.

Meaning:

1. The browser first downloads a **lazy chunk** (the JS file):
    
    ```
    34.chunk.js (GET)
    ```
    
    This file contains the component or code that performs the fetch request.
    
2. Only _after_ the chunk is downloaded and executed does the app know:
    
    > “Oh! I need to fetch some Pokémon data.”
    
3. So now it triggers the actual data request:
    
    ```
    graphql-pokemon2.vercel.app (POST)
    ```
    
4. Once the data arrives, the component renders, which then triggers loading of images:
    
    ```
    pikachu.jpg (GET)
    fallback-pokemon.jpg (GET)
    ```
    
    ...also **after** the data is fetched.
    

---

# 📉 **Why is this bad? (Waterfall = slower app)**

A waterfall means every resource waits for the previous one to finish:

```
Load JS chunk → wait
Then fetch data → wait
Then load images → wait
```

Nothing overlaps. This increases Total Page Load Time.

You ideally want **parallel loading**, like:

```
JS loads    ┐
Data loads  ├── at the same time
Images load ┘
```

But when the component that performs the fetch is lazy-loaded, the browser **can’t start the fetch early**, because:

- Until the JS chunk is downloaded…
    
- The code that calls `fetch()` doesn’t even exist yet.
    

---

# 🧩 **What causes this waterfall?**

Typical situations:

### ❌ When using `React.lazy()`:

```js
const PokemonPage = React.lazy(() => import("./PokemonPage"));
// PokemonPage fetches data inside it
```

### ❌ When using `Next.js dynamic()`:

```js
const PokemonGrid = dynamic(() => import('@/components/PokemonGrid'), {
    loading: () => <Spinner />,
});
```

### ❌ When you fetch **inside** the lazy component:

```js
useEffect(() => {
  fetch(...)
}, []);
```

### Result:

The fetch cannot happen **until after** the JS chunk for the lazy component loads → waterfall.

---

# 🟢 **What’s the recommended fix?**

### ⭐ **Move data fetching OUT of the lazy component.**

Instead of:

```js
const PokemonGrid = React.lazy(() => import('./PokemonGrid'));

function App() {
  return <PokemonGrid />;
}
```

Do:

```js
async function App() {
  const data = await fetchPokemon();       // fetch early
  const PokemonGrid = await import('./PokemonGrid');
  return <PokemonGrid.default data={data} />;
}
```

Or in React 18 / Next.js 13+:

- Use **Server Components**
    
- Use **loaders**
    
- Or fetch data on the server so it’s available _before_ hydration
    

So now:

```
fetch data (first!) → load component → render → load images
```

Much faster. No waterfall.

---

# 📌 Summary

### ✔ Lazy-loading a component delays:

- Loading the JS
    
- Which delays fetching the data
    
- Which delays rendering
    
- Which delays loading images
    

### ✔ These delays stack into a **waterfall**.

That’s why the network bars are staggered.

### ✔ Solution:

Fetch data **outside** the lazy-loaded component so the browser can start earlier and avoid the waterfall.