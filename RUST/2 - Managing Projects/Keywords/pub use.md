---
tags: rust, keyword, fundamental
---

- When we bring a name into scope with the `use` keyword, the name is private to the scope into which we imported it. To enable code outside that scope to refer to that name as if it had been defined in that scope, we can combine `pub` and `use`. This technique is called _re-exporting_ because we’re bringing an item into scope but also making that item available for others to bring into their scope.