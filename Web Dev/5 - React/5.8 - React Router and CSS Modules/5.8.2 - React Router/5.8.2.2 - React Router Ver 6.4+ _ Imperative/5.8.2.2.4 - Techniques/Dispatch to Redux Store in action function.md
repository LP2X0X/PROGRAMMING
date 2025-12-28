---
tags: 
 - react
 - router
 - technique
---

```ad-question
We can not use the dispatch function from useDispatch because useDispatch is a hook. Then how can we dispatch an action to the store inside an action function of React Router?
```

---

````ad-Answer
We can import the store directly and use it inside the action function:
```js
export async function action({ request }) {
  const formData = await request.formData();
  const data = Object.fromEntries(formData);

  const errors = {};
  if (!isValidPhone(data.phone)) {
    errors.phone =
      'Please give us your correct phone number. We might need it to connect you.';
  }

  if (Object.keys(errors).length > 0) return errors;

  const order = {
    ...data,
    cart: JSON.parse(data.cart),
    priority: data.priority === 'true',
  };

  const newOrder = await createOrder(order);
  // DO NOT OVERUSE
  store.dispatch(clearCart());

  return redirect(`/order/${newOrder.id}`);
}
```
Don't overuse this technique because it deactivates a couple of performance optimizations of Redux.
````