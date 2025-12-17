---
tags: 
 - js
 - event
 - method
---

- The fundamental problem of the aforementioned ways to assign handlers is that we _can’t assign multiple handlers to one event_.
- Developers of web standards understood that long ago and suggested an alternative way of managing handlers using the special methods `addEventListener` and `removeEventListener` which aren’t bound by such constraint.

- The syntax to add a handler:

```javascript
element.addEventListener(event, handler, [options]);
```

- `event`: Event name, e.g. `"click"`.
- `handler`: The handler function.
- `options`: An additional optional object with properties:
	- `once`: if `true`, then the listener is automatically removed after it triggers.
	- `capture`: the phase where to handle the event, to be covered later in the chapter [Bubbling and capturing](https://javascript.info/bubbling-and-capturing). For historical reasons, `options` can also be `false/true`, that’s the same as `{capture: false/true}`.
	- `passive`: if `true`, then the handler will not call `preventDefault()`, we’ll explain that later in [Browser default actions](https://javascript.info/default-browser-action).

- To remove the handler, use `removeEventListener`:

```javascript
element.removeEventListener(event, handler, [options]);
```

````ad-note
Removal requires the same function

To remove a handler we should pass exactly the same function as was assigned. Here’s the right way:

```javascript
function handler() {
  alert( 'Thanks!' );
}

input.addEventListener("click", handler);
// ....
input.removeEventListener("click", handler);
````
