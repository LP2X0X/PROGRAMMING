---
tags: 
 - typescript
 - narrowing
---

## What “Control Flow Analysis” means

Control flow analysis is TS’s process of looking at how your code **executes at runtime in terms of branching, return/throw, loops, assignments**, etc., and using that execution-structure _and_ your type annotations to infer more specific (narrowed) types than just the declared ones.  
In other words: TS doesn't only look at type‑guards like `if (typeof x === "string")`, it also watches which code paths are reachable, which assignments happen, whether the function returns early, and so on. The combination of “guards + flow” lets TS narrow.

From the docs:

> “TypeScript was able to analyze this code and see that the rest of the body … is unreachable in the case where `padding` is a `number`. … This analysis of code based on reachability is called control flow analysis.” ([TypeScript](https://www.typescriptlang.org/docs/handbook/2/narrowing.html "TypeScript: Documentation - Narrowing"))

---

## Example from the docs

Consider this function:

```ts
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;
  }
  return padding + input;
}
```

- `padding` starts with type `number | string`.
    
- The `if (typeof padding === "number")` branch is a type‑guard: inside that block TS knows `padding: number`.
    
- Because we `return` inside that branch, that means for the _rest of the function body_ (i.e., after that `if`), the case “padding was number” is **already handled** / unreachable for the “else” part.
    
- So TS **removes** `number` from the type of `padding` in the rest of that function body. In effect it treats `padding` as `string` for the remaining code path.
    
- That’s flow‑analysis: TS uses the fact that `return` happens to know the branch ends and the rest is a different path. ([TypeScript](https://www.typescriptlang.org/docs/handbook/2/narrowing.html "TypeScript: Documentation - Narrowing"))
    

---

## Why this can be really useful

- It lets TS be _smarter_ than just “you wrote `x: A | B`, so I’ll always treat `x` as `A | B` everywhere.” Instead it considers **which branch you’re in**.
    
- It reduces the need for extra type‑assertions (`!`, `as`, etc) because TS knows “okay, in this path `x` can only be this type”.
    
- It helps catch bugs: if you forget to handle a branch, TS may complain or you may get a less narrowed type than you expected.
    

---

## Some other patterns you’ll see via control flow

Here are some additional patterns (and TS uses flow‑analysis + guards for them):

- **Assignments**: when you assign a new value to a variable, TS updates what it “thinks” the type is in that spot. For example, `x = 100` narrows `x` to `number` in that branch. But remember: the declared type is still the umbrella. ([TypeScript](https://www.typescriptlang.org/docs/handbook/2/narrowing.html "TypeScript: Documentation - Narrowing"))
    
- **Early returns / throws**: If you `return` or `throw` inside a branch, TS knows the rest of the function body is a different path.
    
- **Loops / reassignments**: TS tracks that a variable’s type may change depending on loop iterations or assignments.
    
- **Merging/unifying after branches**: After an `if / else` block, TS will “merge” the possible types from both branches for a variable that’s used after.
    
- **Impossible code paths**: If TS can determine a path is unreachable (for example via guards/returns), it may treat types more strictly in reachable parts.
    

---

## Caveats & things to watch out for

- TS’s control flow analysis works _within the function_ (or block) for local variables especially. It’s less strong or more conservative across asynchronous boundaries, closures, or when mutation/capture happens in many places.
    
- If you do weird reassignments or use variables in many places, the narrowing may be less precise (TS will err on the safe side).
    
- Some “non‑type‑guard” code (like custom functions) may not help TS narrow unless you explicitly give a _type predicate_ (e.g., `function isFoo(x): x is Foo { … }`). Without that, TS may not know enough to narrow.
    
- The declared type still matters, as you discovered: even if TS has narrowed a variable in a branch, assignments are still checked against the _declared_ type. (See your earlier example.)
    
- Sometimes you may need to help TS by refactoring so it can better track the flow (reduce reassignments, keep branches clear).
    

---

## How to **apply** this knowledge (tips for you as you code)

Since you said you want the focus to be on React only, here are some tips for how control flow analysis plays out in a React/TypeScript context:

- When you have state or props of a union type (e.g., `foo: string | undefined`), use guards (e.g., `if (foo) { … } else { … }`) so TS knows inside the `if` branch `foo: string`.
    
- When you use callbacks or event handlers, try to keep code paths clear (early returns) so TS can narrow nicely.
    
- Avoid mutating a variable (or state) in many different ways in the same function if you want predictable narrowing. Clear structure = better narrowing.
    
- Use user‑defined type guards when you have complex objects or unions that TS can't naturally infer. That uses the “type predicate” mechanism which is built on control flow analysis.
    
- Be aware that inside React’s render / JSX you’re often writing code that depends on certain props/state being a certain type. Use branching or safe guards so TS knows the correct type in each branch.
    