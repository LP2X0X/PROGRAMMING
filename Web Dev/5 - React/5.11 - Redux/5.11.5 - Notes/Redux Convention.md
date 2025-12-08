---
tags: 
 - react
 - redux
 - convention
 - note
---

- Keep the name of the action creator function identical to the action type name.
  
  ---

- Create the [[Selector Function|selector function]] with the "get" prefix inside the file contain the slice as well:
```js
// Instead of getting the needed property from state like this inside the component function...
const totalCartQuantity = useSelector((state) =>
	state.cart.cart.reduce((sum, item) => sum + item.quantity, 0),
);

// cartSlice.js
// we create the selector function and export it from the "slice file"
export const getTotalCartPrice = (state) =>
  state.cart.cart.reduce((sum, item) => sum + item.quantity, 0);
```

