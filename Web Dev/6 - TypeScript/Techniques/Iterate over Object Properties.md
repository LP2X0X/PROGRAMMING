---
tags: 
 - typescript
 - tecnique
 - object
---

- Either we **strengthen the type of the key** to preserve type safety:
```ts
interface User {
  id: number;
  name: string;
}

function printUser(user: User) {
	Object.keys(user).forEach((key) => console.log(user[key as keyof User]));
}
```

- Or we **loosen the type of the object** to allow more flexible access:
```ts
interface User {
  id: number;
  name: string;
}

function printUser(user: Record<string, any>) {
	Object.keys(user).forEach((key) => console.log(user));
}
```