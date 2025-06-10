- In JavaScript, arrays aren't [primitives](https://developer.mozilla.org/en-US/docs/Glossary/Primitive) but are instead `Array` objects with the following core characteristics:
	- **JavaScript arrays are resizable** and **can contain a mix of different [data types](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures)**. (When those characteristics are undesirable, use [typed arrays](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Typed_arrays) instead.)
	- **JavaScript arrays are not associative arrays** and so, array elements cannot be accessed using arbitrary strings as indexes, but must be accessed using nonnegative integers (or their respective string form) as indexes.
	- **JavaScript arrays are [zero-indexed](https://en.wikipedia.org/wiki/Zero-based_numbering)**: the first element of an array is at index `0`, the second is at index `1`, and so on — and the last element is at the value of the array's [`length`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/length) property minus `1`.
	- **JavaScript [array-copy operations](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array#copy_an_array) create [shallow copies](https://developer.mozilla.org/en-US/docs/Glossary/Shallow_copy)**. (All standard built-in copy operations with _any_ JavaScript objects create shallow copies, rather than [deep copies](https://developer.mozilla.org/en-US/docs/Glossary/Deep_copy)).

---

### Relationship between length and numerical properties
- A JavaScript array's [`length`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/length) property and numerical properties are connected.
- Several of the built-in array methods (e.g., [`join()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/join), [`slice()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice), [`indexOf()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/indexOf), etc.) take into account the value of an array's [`length`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/length) property when they're called.
- Other methods (e.g., [`push()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/push), [`splice()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/splice), etc.) also result in updates to an array's [`length`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/length) property.
```js
const fruits = [];
fruits.push("banana", "apple", "peach");
console.log(fruits.length); // 3
```
</br>

- When setting a property on a JavaScript array when the property is a valid array index and that index is outside the current bounds of the array, the engine will update the array's [`length`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/length) property accordingly:
```
fruits[5] = "mango";
console.log(fruits[5]); // 'mango'
console.log(Object.keys(fruits)); // ['0', '1', '2', '5']
console.log(fruits.length); // 6
```
</br>

- Increasing the [`length`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/length) extends the array by adding empty slots without creating any new elements — not even `undefined`.
```js
fruits.length = 10;
console.log(fruits); // ['banana', 'apple', 'peach', empty x 2, 'mango', empty x 4]
console.log(Object.keys(fruits)); // ['0', '1', '2', '5']
console.log(fruits.length); // 10
console.log(fruits[8]); // undefined
```
</br>

- Decreasing the [`length`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/length) property does, however, delete elements.
```js
fruits.length = 2;
console.log(Object.keys(fruits)); // ['0', '1']
console.log(fruits.length); // 2
```