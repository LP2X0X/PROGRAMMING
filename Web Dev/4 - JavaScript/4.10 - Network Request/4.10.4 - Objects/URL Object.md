---
tags: 
 - js
 - url
 - object
---

The built-in [URL](https://url.spec.whatwg.org/#api) class provides a convenient interface for creating and parsing [[URL|URLs]].

There are no networking methods that require exactly a `URL` object, strings are good enough. So technically we don’t have to use `URL`. But sometimes it can be really helpful.

---

## Creating a URL

The syntax to create a new `URL` object:

```js
new URL(url, [base])
```

- **`url`** – the full URL or only path (if base is set, see below),
- **`base`** – an optional base URL: if set and `url` argument has only path, then the URL is generated relative to `base`.

For example:

```javascript
let url = new URL('https://javascript.info/profile/admin');
```

These two URLs are same:
```js
let url1 = new URL('https://javascript.info/profile/admin');
let url2 = new URL('/profile/admin', 'https://javascript.info');

alert(url1); // https://javascript.info/profile/admin
alert(url2); // https://javascript.info/profile/admin
```

We can easily create a new URL based on the path relative to an existing URL:
```js
let url = new URL('https://javascript.info/profile/admin');
let newUrl = new URL('tester', url);

alert(newUrl); // https://javascript.info/profile/tester
```

The `URL` object immediately allows us to access its components, so it’s a nice way to parse the url, e.g.:
```js
let url = new URL('https://javascript.info/url');

alert(url.protocol); // https:
alert(url.host);     // javascript.info
alert(url.pathname); // /url
```

```ad-note
We can use a `URL` object in `fetch` or `XMLHttpRequest`, almost everywhere where a URL-string is expected.

Generally, the `URL` object can be passed to any method instead of a string, as most methods will perform the string conversion, that turns a `URL` object into a string with full URL.
```

---

## SearchParams “?…”

Let’s say we want to create a url with given search params, for instance, `https://google.com/search?query=JavaScript`.

We can provide them in the URL string:

```js
new URL('https://google.com/search?query=JavaScript')
```

…But parameters need to be encoded if they contain spaces, non-latin letters, etc (more about that below).

So there’s a URL property for that: `url.searchParams`, an object of type URLSearchParams.

It provides convenient methods for search parameters:

- **`append(name, value)`** – add the parameter by `name`,
- **`delete(name)`** – remove the parameter by `name`,
- **`get(name)`** – get the parameter by `name`,
- **`getAll(name)`** – get all parameters with the same `name` (that’s possible, e.g. `?user=John&user=Pete`),
- **`has(name)`** – check for the existence of the parameter by `name`,
- **`set(name, value)`** – set/replace the parameter,
- **`sort()`** – sort parameters by name, rarely needed,
- …and it’s also iterable, similar to `Map`.

An example with parameters that contain spaces and punctuation marks:
```js
let url = new URL('https://google.com/search');

url.searchParams.set('q', 'test me!'); // added parameter with a space and !

alert(url); // https://google.com/search?q=test+me%21

url.searchParams.set('tbs', 'qdr:y'); // added parameter with a colon :

// parameters are automatically encoded
alert(url); // https://google.com/search?q=test+me%21&tbs=qdr%3Ay

// iterate over search parameters (decoded)
for(let [name, value] of url.searchParams) {
  alert(`${name}=${value}`); // q=test me!, then tbs=qdr:y
}
```

---
### Reference

[[URL]]