---
tags: vim, trick
---

In **Vim**, you can **quickly paste the word under the cursor into command mode** using the following trick:

---

### ✅ Method: Use `Ctrl-R` + `"` in Command Mode

1. While **in Normal mode**, move the cursor to the word you want.
2. Press `:` to enter **Command mode**.
3. Then type:

   ```vim
   :<Ctrl-R><Ctrl-W>
   ```

   * `Ctrl-R` tells Vim: “insert the contents of a register”
   * `Ctrl-W` means: “the word under the cursor”

> This will paste the **word under the cursor** into the command line where your cursor is.

---

### 🔄 Example Use Case

Say you want to search for the word under the cursor:

1. Place cursor on `exampleWord`
2. Press `:` to enter command mode
3. Type `/` then `<Ctrl-R><Ctrl-W>` → this becomes:

   ```vim
   :/exampleWord
   ```

---

### 📝 Bonus: Yank word under cursor

You can also yank the word under the cursor with:

```vim
"ayiw
```

Then paste it with `Ctrl-R a` in command mode.