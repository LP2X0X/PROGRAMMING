---
tags: js, keyword, fundamental
---

- The **instanceof** keword helps test whether an object is an instance of a particular [[Web Dev/3 - JavaScript Crash Course/1 - Basic/Classes/Class|class]].

```js
class Vehicle
{}

class Car extends Vehicle
{}

let car = new Car();

car instanceof Vehicle;
-> true
car instanceof Car;
-> true
```