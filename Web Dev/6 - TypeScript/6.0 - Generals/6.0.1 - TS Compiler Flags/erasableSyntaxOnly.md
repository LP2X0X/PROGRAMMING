---
tags: 
 - typescript
 - compiler
 - flag
---

**Erasable-syntax-only** means:

> You may use **only TypeScript syntax that can be completely deleted** without changing the program’s runtime behavior.

### Allowed

Type-only constructs that vanish cleanly:

- Type annotations (`: number`)
    
- Interfaces / type aliases
    
- Generics
    
- `as` assertions
    
- `implements`
    

### Not allowed

Anything that would produce or alter runtime code:

- `enum`
    
- Parameter properties (`constructor(public x: number)`)
    
- Decorators
    
- Namespaces
    
- `abstract`, `override`
    

### One-line rule

> If deleting the TypeScript syntax leaves valid JavaScript with the same behavior, it’s allowed; otherwise, it isn’t.