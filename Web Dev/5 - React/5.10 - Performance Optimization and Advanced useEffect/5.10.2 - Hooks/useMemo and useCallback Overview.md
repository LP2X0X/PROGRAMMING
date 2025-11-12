---
tags: 
 - react
 - memoization
 - overview
---

![[Pasted image 20251108205035.png|center|700]]
![[Pasted image 20251108205348.png|center|700]]

---

### Example of infinite useEffect loop
- For example, in a context provider component, we have this function:
```js  
async function getCity(id) {
    if (Number(id) === currentCity) {
      return;
    }

    try {
      dispatch({ type: "loading" });
      const res = await fetch(`${BASE_URL}/cities/${id}`);
      if (!res.ok) {
        throw new Error("There is something wrong when fetching city data!");
      }
      const data = await res.json();
      dispatch({ type: "city/loaded", payload: data });
    } catch (err) {
      dispatch({ type: "rejected", payload: err.message });
    }
  }
```

- In some components, we access a function from the context and use it within an effect:
```js
const { getCity, currentCity, isLoading } = useCities();

useEffect(function () {
  getCity(id);
}, [id, getCity]);
```

- Now, here's the problem: getCity update the state inside the context each time that it is executed which in turn cause the context provider to re-render. Therefore, a new getCity function is created. Since a new getCity is created and also it is in the dependency array of the above useEffect, getCity will be called again. In conclusion, it will cause an **infinite loop**.