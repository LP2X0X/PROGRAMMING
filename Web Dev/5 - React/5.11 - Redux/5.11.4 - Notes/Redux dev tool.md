---
tags: 
 - react
 - redux
 - dev
 - tool
 - note
---

I’ll give two recommended setups:

- **Preferred:** using **Redux Toolkit (`configureStore`)** — simplest, devtools enabled automatically. ([redux-toolkit.js.org](https://redux-toolkit.js.org/api/configureStore?utm_source=chatgpt.com "configureStore"))
    
- **Manual / legacy:** plain Redux store + `@redux-devtools/extension` (`composeWithDevTools`) — good if you’re not using RTK. ([npm](https://www.npmjs.com/package/%40redux-devtools/extension?utm_source=chatgpt.com "redux-devtools/extension"))
    

Also: make sure you have the **browser extension** installed (Chrome/Firefox). ([Chrome Web Store](https://chromewebstore.google.com/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd?hl=en&utm_source=chatgpt.com "Redux DevTools - Chrome Web Store"))

---

# Quick checklist (what you’ll do)

1. Install browser Redux DevTools extension. ([Chrome Web Store](https://chromewebstore.google.com/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd?hl=en&utm_source=chatgpt.com "Redux DevTools - Chrome Web Store"))
    
2. If using RTK: no extra package required; `configureStore` enables devtools. ([redux-toolkit.js.org](https://redux-toolkit.js.org/api/configureStore?utm_source=chatgpt.com "configureStore"))
    
3. If using plain Redux: install `@redux-devtools/extension` and use `composeWithDevTools`. ([npm](https://www.npmjs.com/package/%40redux-devtools/extension?utm_source=chatgpt.com "redux-devtools/extension"))
    
4. Optional: only enable in development. (examples below)
    

---

# A. Install the browser extension

1. Chrome: open Chrome Web Store → **Redux DevTools** → Install.
    
2. Firefox: install the Redux DevTools add-on.  
    (After install you’ll see the Redux icon in the toolbar and a “Redux” panel in DevTools). ([Chrome Web Store](https://chromewebstore.google.com/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd?hl=en&utm_source=chatgpt.com "Redux DevTools - Chrome Web Store"))
    

---

# B. Preferred: If you use **Redux Toolkit** (`configureStore`)

Redux Toolkit configures DevTools automatically (development only) and wires up thunk middleware by default.

1. Install Redux Toolkit (if you don’t already):
    

```bash
npm install @reduxjs/toolkit react-redux
```

2. Create the store with `configureStore`:
    

```js
// src/store.js
import { configureStore } from '@reduxjs/toolkit'
import rootReducer from './rootReducer' // or pass your slice reducers

const store = configureStore({
  reducer: rootReducer,
  // devTools is true by default in dev; you can pass options or false to disable
  devTools: process.env.NODE_ENV !== 'production',
})

export default store
```

3. Use `<Provider store={store}>` in your React root as usual.
    

Why this is easiest: `configureStore` automatically sets up the Redux DevTools connection and common middleware. ([redux-toolkit.js.org](https://redux-toolkit.js.org/api/configureStore?utm_source=chatgpt.com "configureStore"))

---

# C. Manual (plain Redux) — `@redux-devtools/extension` (recommended over old `redux-devtools-extension`)

> Use this if you create your store manually (applyMiddleware / createStore).  
> Note: `redux-devtools-extension` historically required Redux 3/4; for Redux v5 use `@redux-devtools/extension`. (That’s the compatibility fix you saw.) ([Stack Overflow](https://stackoverflow.com/questions/78013328/there-are-problem-when-i-install-redux-devtools-extension?utm_source=chatgpt.com "there are problem when i install redux-devtools-extension"))

1. Install the package:
    

```bash
npm install @redux-devtools/extension redux redux-thunk react-redux
```

2. Wire it into your store file:
    

```js
// src/store.js
import { createStore, applyMiddleware } from 'redux'
import thunk from 'redux-thunk'
import rootReducer from './rootReducer'
import { composeWithDevTools } from '@redux-devtools/extension'

const middleware = [thunk]

// enable devtools only in development
const composeEnhancers = composeWithDevTools({
  // options like trace: true can go here
  trace: true,
  traceLimit: 25,
})

const store = createStore(
  rootReducer,
  composeEnhancers(applyMiddleware(...middleware))
)

export default store
```

3. Wrap your app with `<Provider store={store}>`.
    

If you prefer a minimal enhancer without options:

```js
const store = createStore(rootReducer, composeWithDevTools(applyMiddleware(...middleware)));
```

---

# D. Example: only enable DevTools in development

If you want more control (or to avoid shipping devtools code to production):

```js
import { composeWithDevTools } from '@redux-devtools/extension'

const enhancer =
  process.env.NODE_ENV === 'production'
    ? applyMiddleware(...middleware)
    : composeWithDevTools({ trace: true })(applyMiddleware(...middleware))

const store = createStore(rootReducer, enhancer)
```

Or with RTK:

```js
configureStore({ reducer, devTools: process.env.NODE_ENV !== 'production' })
```

---

# E. Common troubleshooting & tips

- **Peer dependency conflict with Redux 5:** if you tried to install `redux-devtools-extension` and got `ERESOLVE` complaining about peer redux `^3.1.0 || ^4.0.0`, that package expects Redux 3/4. Use `@redux-devtools/extension` instead (supports modern Redux). ([Stack Overflow](https://stackoverflow.com/questions/78013328/there-are-problem-when-i-install-redux-devtools-extension?utm_source=chatgpt.com "there are problem when i install redux-devtools-extension"))
    
- **Extension shows “no store” (greyed out):**
    
    - Ensure your app actually created a Redux store before the page loaded.
        
    - Make sure the extension has site access (browser extension permissions). ([Stack Overflow](https://stackoverflow.com/questions/60838038/how-to-enable-redux-devtools?utm_source=chatgpt.com "How to enable Redux Devtools?"))
        
- **If using server-side rendering:** make sure devtools are only enabled in client-side code (guard by `typeof window !== 'undefined'` or `process.env.NODE_ENV`).
    
- **Redux Toolkit recommended:** it reduces boilerplate and sets up devtools for you automatically. If you can, prefer RTK. ([redux-toolkit.js.org](https://redux-toolkit.js.org/api/configureStore?utm_source=chatgpt.com "configureStore"))
    

---

# F. Quick verification

1. Start your app (`npm start`) and open browser DevTools → **Redux** tab (or click Redux icon).
    
2. You should see actions appear as you dispatch them, the state tree, time-travel playback, action diffing, etc.
    

---

# G. Want me to:

- write the exact `store.js` for your repo (you can paste your rootReducer)?
    
- convert your existing createStore code to RTK `configureStore`?  
    If yes, paste your current `store.js` or `rootReducer` and I’ll produce the ready-to-use file.
    

---

### Sources / further reading

- Redux docs — configuring store & DevTools with configureStore. ([redux-toolkit.js.org](https://redux-toolkit.js.org/api/configureStore?utm_source=chatgpt.com "configureStore"))
    
- `@redux-devtools/extension` (npm) — recommended package for modern setups. ([npm](https://www.npmjs.com/package/%40redux-devtools/extension?utm_source=chatgpt.com "redux-devtools/extension"))
    
- Discussion / Q&A about `redux-devtools-extension` vs `@redux-devtools/extension` (compatibility). ([Stack Overflow](https://stackoverflow.com/questions/78013328/there-are-problem-when-i-install-redux-devtools-extension?utm_source=chatgpt.com "there are problem when i install redux-devtools-extension"))
    

Would you like a ready-to-paste `store.js` matched to your code (vanilla or RTK)? Paste your current `rootReducer` and I’ll generate it.