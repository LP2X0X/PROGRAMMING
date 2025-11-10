---
tags: 
 - react
 - optimization
 - children
---

- Here's an example to visualize things:
```jsx
import { useState } from "react";

// We got a slow and expensive component
function SlowComponent() {
  const words = Array.from({ length: 100_000 }, () => "WORD");
  return (
    <ul>
      {words.map((word, i) => (
        <li key={i}>
          {i}: {word}
        </li>
      ))}
    </ul>
  );
}

// And a parent component which contain it as a child component
export default function Test() {
  const [count, setCount] = useState(0); // Everytime this state change
  // These below get re-render
  return (
    <div>
      <h1>Slow counter?!?</h1>
      <button onClick={() => setCount((c) => c + 1)}>Increase: {count}</button>

      <SlowComponent />
    </div>
  );
}
```

- Here's how to optimize it:
	- Instead of treating it like a part of the parent component it self, we **pass it as a children prop** (1).
	- Then we use it inside the parent component like usual (2).
```jsx
import { useState } from "react";

function SlowComponent() {
  const words = Array.from({ length: 100_000 }, () => "WORD");
  return (
    <ul>
      {words.map((word, i) => (
        <li key={i}>
          {i}: {word}
        </li>
      ))}
    </ul>
  );
}

function Counter({ children }) {
  const [count, setCount] = useState(0);
  return (
    <div>
      <h1>Slow counter?!?</h1>
      <button onClick={() => setCount((c) => c + 1)}>Increase: {count}</button>

	  /* (2) */
      {children}
    </div>
  );
}

export default function Test() {
  // const [count, setCount] = useState(0);
  // return (
  //   <div>
  //     <h1>Slow counter?!?</h1>
  //     <button onClick={() => setCount((c) => c + 1)}>Increase: {count}</button>

  //     <SlowComponent />
  //   </div>
  // );

  return (
    <div>
      <h1>Slow counter?!?</h1>
      <Counter>
	     /* (1) */
        <SlowComponent />
      </Counter>
    </div>
  );
}
```

```ad-Answer
This work because Test component (and therefore the SlowComponent component), has already been created before the Counter component re-rendered.
Therefore, there is no way in which SlowComponent component could have been affected by the state change in the Counter (it has already been passed in).
```