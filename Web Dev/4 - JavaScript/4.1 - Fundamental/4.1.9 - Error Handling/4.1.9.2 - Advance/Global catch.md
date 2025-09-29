---
tags: 
 - js
 - error
 - handle
 - global
---

Let’s imagine we’ve got a fatal error outside of `try...catch`, and the script died. Like a programming error or some other terrible thing.

Is there a way to react on such occurrences? We may want to log the error, show something to the user (normally they don’t see error messages), etc.

There is none in the specification, but environments usually provide it, because it’s really useful. For instance, Node.js has [`process.on("uncaughtException")`](https://nodejs.org/api/process.html#process_event_uncaughtexception) for that. And in the browser we can assign a function to the special [window.onerror](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onerror) property, that will run in case of an uncaught error.

The syntax:

```javascript
window.onerror = function(message, url, line, col, error) {
  // ...
};
```

`message`

Error message.

`url`

URL of the script where error happened.

`line`, `col`

Line and column numbers where error happened.

`error`

Error object.

For instance:

```markup
<script>
  window.onerror = function(message, url, line, col, error) {
    alert(`${message}\n At ${line}:${col} of ${url}`);
  };

  function readData() {
    badFunc(); // Whoops, something went wrong!
  }

  readData();
</script>
```

The role of the global handler `window.onerror` is usually not to recover the script execution – that’s probably impossible in case of programming errors, but to send the error message to developers.

There are also web-services that provide error-logging for such cases, like [https://muscula.com](https://muscula.com) or [https://www.sentry.io](https://www.sentry.io).

They work like this:

1. We register at the service and get a piece of JS (or a script URL) from them to insert on pages.
2. That JS script sets a custom `window.onerror` function.
3. When an error occurs, it sends a network request about it to the service.
4. We can log in to the service web interface and see errors.