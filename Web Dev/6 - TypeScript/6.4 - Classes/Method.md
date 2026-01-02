---
tags: 
 - typescript
 - class
 - method
---

A function property on a class is called a method. Methods can use all the same [[Function Type Annotation|type annotations]] as functions and constructors:

```ts
class Point {
  x = 10;
  y = 10;
 
  scale(n: number): void {
    this.x *= n;
    this.y *= n;
  }
}
```

Other than the standard type annotations, TypeScript doesn’t add anything else new to methods.

Note that inside a method body, it is still mandatory to access fields and other methods via `this.`. An unqualified name in a method body will always refer to something in the enclosing scope:

```ts
let x: number = 0;
 
class C {
  x: string = "hello";
 
  m() {
    // This is trying to modify 'x' from line 1, not the class property
    x = "world";
Type 'string' is not assignable to type 'number'.

  }
}
```

---

## Using arrow function 

When the instance is created, the arrow function is created **with `this` fixed to that instance**.

```ts
class CanvasNode {
	x = 0;
	y = 0;
	
	move = (x: number, y: number) {
		this.x = x;
		this.y = y;
	}
}
```