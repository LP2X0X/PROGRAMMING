---
tags: js, datatype, fundamental
---

- A JavaScript array is a compound data type that holds an ordered list of values.

```ad-important
The elements of an array can be of any data type.
They have the flexibility to grow and shrink as values are added or removed.
```

---

### Creation and Indexing
- To create an array, list its elements separated by commas inside a pair of square brackets:
  
```js
let primes = [2, 3, 5, 7, 11, 13, 17, 19];
```
- To access an individual element of an array, place its index number in square brackets after the name of the array.

```js
primes[0];
-> 2
```

- To replace an element in an array, assign the element a new value using indexing syntax:

```js
primes[2] = 1;
```
