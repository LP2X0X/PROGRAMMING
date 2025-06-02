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

---

### Array methods

Here’s a **summary of commonly used JavaScript array methods**—organized by purpose:

---

### **🔁 Iteration**

|**Method**|**Description**|
|---|---|
|forEach(callback)|Executes a function for each element (no return).|
|map(callback)|Transforms each element and returns a new array.|
|filter(callback)|Returns a new array with elements that pass a test.|
|reduce(callback, initial)|Reduces array to a single value.|
|some(callback)|Returns true if **any** element passes the test.|
|every(callback)|Returns true if **all** elements pass the test.|
|find(callback)|Returns the **first** matching element.|
|findIndex(callback)|Returns the **index** of the first match.|

---

### **🔄 Modify / Mutate**

|**Method**|**Description**|
|---|---|
|push(item)|Add item(s) to the **end**.|
|pop()|Remove and return item from the **end**.|
|unshift(item)|Add item(s) to the **start**.|
|shift()|Remove and return item from the **start**.|
|splice(start, deleteCount, ...items)|Add/remove items at index.|
|sort()|Sorts the array (can be custom with a function).|
|reverse()|Reverses the array in-place.|
|fill(value, start, end)|Fills array with a value.|
|copyWithin(target, start, end)|Copies a portion within the same array.|

---

### **📋 Non-Mutating / Utility**

|**Method**|**Description**|
|---|---|
|concat()|Combines arrays (returns new array).|
|slice(start, end)|Returns a shallow copy (sub-array).|
|includes(value)|Checks if value exists in the array.|
|indexOf(value)|Returns index of first match, or -1.|
|lastIndexOf(value)|Index of last occurrence.|
|join(separator)|Converts array to string with a separator. Default separator is comma.|
|toString()|Converts array to comma-separated string.|
|at(index)|Returns element at index (supports negative index).|

---

### **🆕 ES2023+**

| **Method**         | **Description**                         |
| ------------------ | --------------------------------------- |
| toReversed()       | Returns a reversed copy (non-mutating). |
| toSorted()         | Returns a sorted copy (non-mutating).   |
| toSpliced()        | Returns spliced copy (non-mutating).    |
| with(index, value) | Returns new array with replaced value.  |

---

### Others
```ad-note
If you use entries() method of an array, it will return an array of arrays where each inner array contains the index as key and the item as value.
```