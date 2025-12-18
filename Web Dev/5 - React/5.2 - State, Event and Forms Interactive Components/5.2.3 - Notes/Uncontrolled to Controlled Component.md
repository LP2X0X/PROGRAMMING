---
tags: 
 - react
 - note
 - controlled
 - component
---

```js
import React from 'react';

export default class MyForm extends React.Component {
    constructor(props) {
        super(props);
        this.state = {}
    }

    render() {
        return (
            <form className="add-support-staff-form">
                <input name="name" type="text" value={this.state.name} onChange={this.onFieldChange('name').bind(this)}/>
            </form>
        )
    }

    onFieldChange(fieldName) {
        return function (event) {
            this.setState({[fieldName]: event.target.value});
        }
    }
}

export default MyForm;
```

- For an input to be controlled, its value must correspond to that of a state variable.

- That condition is not initially met in your example because `this.state.name` is not initially set. Therefore, the input is initially uncontrolled. Once the `onChange` handler is triggered for the first time, `this.state.name` gets set. At that point, the above condition is satisfied and the input is considered to be controlled. This transition from uncontrolled to controlled produces the error seen above.

- By initializing `this.state.name` in the constructor:

```javascript
this.state = { name: '' };
```

the input will be controlled from the start, fixing the issue. See [React Controlled Components](https://facebook.github.io/react/docs/forms.html#controlled-components "Controlled Components") for more examples.

---

### Example pitfalls

```jsx
<input
  type="hidden"
  name="position"
  value={
    position.latitude &&
    position.longitude &&
    `${position.latitude}, ${position.longitude}`
  }
/>
```

At first glance this looks harmless, but it triggers React’s  
**“A component is changing an uncontrolled input to be controlled”** warning.

Here is what happens:

1. On the first render, `position.latitude` or `position.longitude` may be `undefined`.
    
2. Because the `&&` expression fails, the whole expression evaluates to **`undefined`**.
    
3. That means the `<input>` initially has **no value attribute** → it is **uncontrolled**.
    
4. Later, when `position` becomes available, the expression returns a **string**, so the input **now has a value** → it becomes **controlled**.
    
5. Switching from uncontrolled → controlled triggers the React warning.
    

#### **How to Fix It**

To avoid the warning, React requires that the input’s `value` **always** be a string from the start.

So instead of letting the value become `undefined`, return an empty string as the default:

```jsx
<input
  type="hidden"
  name="position"
  value={
    position.latitude && position.longitude
      ? `${position.latitude}, ${position.longitude}`
      : ""
  }
/>
```

This guarantees:

- On the initial render → `value=""` (controlled by the `position` state because its value is derived directly from that state)
    
- Once coordinates exist → `value="lat, lng"` (still controlled)
    

React is satisfied because the input is **consistently controlled**.

---

### Reference
[[Controlled Components]]
https://stackoverflow.com/questions/37427508/react-changing-an-uncontrolled-input