---
tags: js, term, advance
---

- Before JavaScript is published to production, it’s compressed using a _minifier_ – a special program that shrinks code by removing extra comments, spaces and – what’s important, renames local variables into shorter ones.
- For instance, if a function has `let userName`, minifier replaces it with `let a` (or another letter if this one is occupied), and does it everywhere. That’s usually a safe thing to do, because the variable is local, nothing outside the function can access it. And inside the function, minifier replaces every mention of it. Minifiers are smart, they analyze the code structure, so they don’t break anything. They’re not just a dumb find-and-replace.