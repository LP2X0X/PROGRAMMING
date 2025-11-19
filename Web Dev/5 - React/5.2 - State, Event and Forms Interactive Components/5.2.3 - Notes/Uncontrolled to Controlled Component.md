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

### Reference
https://stackoverflow.com/questions/37427508/react-changing-an-uncontrolled-input