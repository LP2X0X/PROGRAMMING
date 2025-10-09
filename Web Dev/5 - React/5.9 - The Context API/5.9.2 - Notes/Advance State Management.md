---
tags: 
 - react
 - state
 - management
 - advance
---

## Prerequisite
[[State Management]]

![[Pasted image 20251008171844.png|center|700]]

---

## State Placement Options

We have 6 different options to where we can place a state:

![[Pasted image 20251008203554.png|center|600]]

### 🏠 **Local Component State**

**Tool:**  
`useState`, `useReducer`, or `useRef`

**When to use:**  
When the state is **only needed inside one component** — not shared elsewhere.

**Examples:**

- A form input value in a single component
    
- A modal open/close toggle
    
- A counter
    
- A hover or focus effect
    

**Why use it:**

- Simpler and faster
    
- Avoids unnecessary re-renders in other parts of the app
    
- Keeps logic isolated
    

**Example:**

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

---

### 👨‍👩‍👧 **Parent Component (Lifting State Up)**

**Tool:**  
`useState`, `useReducer`, or `useRef`

**When to use:**  
When **multiple child components need to share** or **synchronize** the same state.

**Examples:**

- A form wizard where different steps share data
    
- A search bar (parent) updating a result list (child)
    
- Managing a selected item between a list and details panel
    

**Why use it:**

- Avoids duplicate state
    
- Keeps related components consistent
    
- Simple alternative before reaching for Context
    

**Example:**

```jsx
function App() {
  const [query, setQuery] = useState('');
  return (
    <>
      <SearchBox value={query} onChange={setQuery} />
      <Results query={query} />
    </>
  );
}
```

---

### 🌐 **Context (React Context API)**

**Tool:**  
`Context API` + `useState` or `useReducer`

**When to use:**  
For **global state** that many components need, but not large or remote data.  
Best for **UI-related global state**, like theme, language, or authentication.

**Examples:**

- Dark/light theme toggling
    
- Current user session
    
- Modal visibility across app
    
- App-wide filter or layout settings
    

**Why use it:**

- Avoids _prop drilling_
    
- Clean, declarative global state
    
- Easy to combine with custom hooks
    

**Example:**

```jsx
const ThemeContext = createContext();

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}
```

---

### ⚙️ **3rd-Party Library**

**Tool:**  
Redux, React Query, SWR, Zustand, Jotai, Recoil, etc.

**When to use:**  
When state is **complex**, involves **server data**, or needs **performance optimization**.  
Can manage **remote or local global state**.

**Examples:**

- Fetching and caching API data (React Query)
    
- Large-scale app with multiple interacting states (Redux, Zustand)
    
- Offline-first data persistence
    

**Why use it:**

- Better for scaling large apps
    
- Easier async handling
    
- Dev tools and middlewares
    

**Example (Redux-like):**

```jsx
import { create } from 'zustand';

const useStore = create((set) => ({
  count: 0,
  increase: () => set((state) => ({ count: state.count + 1 })),
}));
```

---

### 📍 **URL (React Router)**

**Tool:**  
React Router (with `useSearchParams`, `useParams`, `useNavigate`, etc.)

**When to use:**  
When the state should be **shareable via URL**, or **affect navigation**.  
It represents **global state tied to routing**.

**Examples:**

- Current page number (`?page=2`)
    
- Selected filter (`?category=books`)
    
- Dynamic routes (`/user/:id`)
    
- Tabs or modals represented by the URL
    

**Why use it:**

- Keeps UI state in sync with browser navigation
    
- Enables deep linking and browser back/forward support
    
- Makes app behavior shareable/bookmarkable
    

**Example:**

```jsx
const [searchParams, setSearchParams] = useSearchParams();
const page = searchParams.get('page') || 1;
```

---

### 🧭 **Browser Storage**

**Tool:**  
`localStorage`, `sessionStorage`, `IndexedDB`, `cookies`

**When to use:**  
When you need to **persist data** across sessions or page reloads.

**Examples:**

- User preferences (theme, language)
    
- Tokens or login data (non-sensitive)
    
- Draft data or cart items
    

**Why use it:**

- Keeps state available after refresh
    
- Complements React state for persistence
    
- Useful for caching lightweight data
    

**Example:**

```jsx
useEffect(() => {
  localStorage.setItem('theme', theme);
}, [theme]);
```

---

## ✅ Summary Table

| State Location        | Tool                               | When to Use                        | Example                   |
| --------------------- | ---------------------------------- | ---------------------------------- | ------------------------- |
| **Local Component**   | `useState`, `useReducer`, `useRef` | For internal component logic       | Counter, modal open/close |
| **Parent Component**  | `useState`, `useReducer`           | When multiple children share state | Search bar + results      |
| **Context**           | Context API + Hook                 | Global UI state                    | Theme, auth, language     |
| **3rd-Party Library** | Redux, Zustand, React Query        | Complex or remote data             | API caching, large apps   |
| **URL**               | React Router                       | Navigation or shareable state      | `?page=2`, `/user/5`      |
| **Browser**           | `localStorage`, `sessionStorage`   | Persist across reloads             | Theme, cart, token        |

----

### State Placement Tool Option
![[Pasted image 20251009094511.png|center|600]]