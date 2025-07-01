---
tags: react, summary
---

🔷 What is React?

- React (or React.js/ReactJS) is a JavaScript library for building user interfaces.
- Developed and maintained by Facebook.
- Component-based, declarative, and efficient.
- Often used to build single-page applications (SPA).

🔧 Core Concepts

1. ✅ Components
- Basic building blocks of React UI.
- Two types: Functional (Stateless) and Class components (deprecated for most use cases).
- Components accept props and can manage state.

2. 🎛 Props (Short for "Properties")
- Read-only input passed from parent to child components.
- Used to customize and reuse components.

3. 🧠 State
- Local data storage in a component that can be changed with user interaction, API responses, etc.
- Changing state causes the component to re-render.

4. 🔁 Lifecycle Methods (in Class Components)
- Mounting: constructor, render, componentDidMount
- Updating: componentDidUpdate
- Unmounting: componentWillUnmount
- In functional components, useEffect hook replaces these.

5. ⚙ Virtual DOM
- React creates a lightweight copy of the real DOM.
- It uses diffing algorithms to update only the parts that change.

6. 🎣 Hooks (since React 16.8)
- Functions that let you "hook into" React features in functional components.
- useState, useEffect, useContext, useRef, useMemo, useReducer, etc.

7. 🌐 JSX (JavaScript XML)
- Syntax extension that looks like HTML inside JavaScript.
- Allows mixing UI and logic.
Example:
```jsx
function Hello() {
  return <h1>Hello, world!</h1>
}
```

🚀 Additional Features

8. 🔃 Controlled vs Uncontrolled Components
- Controlled: component manages form values via state.
- Uncontrolled: DOM manages state via refs.

9. 🧭 React Router
- Handles routing/navigation in SPAs.
- Example: <Route />, <Link />, useNavigate()

10. ⚛ Context API
- Used for global state management (like Redux lite).
- Avoids prop-drilling.

11. ♻️ useEffect
- Handles side effects (e.g., API calls, subscriptions).
- Replaces lifecycle methods in functional components.

12. 🎯 Keys
- Unique identifiers used in lists to help React identify which items changed.

13. 📦 State Management (External)
- Context API (built-in)
- Redux, Zustand, Jotai, MobX (popular state managers)

14. 👉 Forms
- Controlled forms using inputs with value and onChange
- Libraries: Formik, React Hook Form

🌍 Popular Tools & Ecosystem Around React

- Next.js (React + SSR + routing + API handling)
- Vite (fast dev build tool for React)
- React Native (for mobile app development)
- Tailwind CSS (utility-first CSS with React)
- Material UI, Ant Design, Chakra UI (component libraries)
- Axios / Fetch (for APIs)

📦 Typical React App Directory

```
/src
  /components
  /pages
  App.js
  index.js
```

📁 Common Files:

- App.js – main app layout
- index.js – entry point rendering <App /> with ReactDOM

🛠 Build Tools

- Create React App (CRA): Official boilerplate (now eclipsed by Vite, Next.js)
- Vite: Fast lightweight alternative to CRA
- Webpack + Babel: Manual setup (for advanced users)

🌈 Pros of React

- Component-based & reusable
- Fast with virtual DOM
- Large community and ecosystem
- Works well with modern JS

⚠️ Cons

- JSX may be weird at first
- Lacks built-in routing or state management (have to use libraries)
- Frequent updates and new patterns (hooks, suspense, concurrent)

🌐 Example: Basic App with State and Event

```jsx
import React, { useState } from 'react'

function Counter() {
  const [count, setCount] = useState(0)

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  )
}
```

🆕 Latest Features (React 18+)

- Concurrent Mode (improves rendering performance)
- useTransition, useDeferredValue — for optimizing UI
- Automatic Batching: multiple state updates grouped into one render.
- Server Components (React 19 preview)

📦 Getting Started

To start a new React project:

Via Create React App:
```bash
npx create-react-app my-app
cd my-app
npm start
```

Via Vite:
```bash
npm create vite@latest my-app --template react
cd my-app
npm install
npm run dev
```

📚 Learning Resources

- reactjs.org — Official docs
- Beta/Next docs: react.dev
- React GitHub Repo: github.com/facebook/react
- freeCodeCamp, Codecademy, Frontend Masters, etc.

🧠 In One Sentence:

React is a powerful, declarative JavaScript library for building modern, fast, dynamic user interfaces—based around reusable components and scalable state management.

—

Let me know if you'd like a full cheat sheet, advanced topics (like memoization, concurrency, custom hooks), or help learning through projects!