In Vim's Ex command mode (accessed by typing `:`), the `:normal` command allows you to execute **Normal mode** commands as if you were in Normal mode. This is useful when you want to run a Normal mode command over a range of lines or automate tasks in scripts.

### Basic Syntax:
```
:normal {commands}
```

- The command inside `:normal` is executed as if you typed it in Normal mode.

### Examples:

1. **Delete the current line:**
   ```vim
   :normal dd
   ```
   Equivalent to typing `dd` in Normal mode to delete the current line.

2. **Move the cursor to the end of the line (`$`) for the current line:**
   ```vim
   :normal $
   ```

3. **Insert a new line below the current line and enter Insert mode (`o`):**
   ```vim
   :normal o
   ```

### Executing Normal Commands Over a Range:

- You can combine ranges with the `:normal` command to execute Normal mode commands on multiple lines.

**Example:**
- Delete the first character of every line from lines 1 to 10:
  ```vim
  :1,10normal ^
  ```
  The command `^` moves to the first non-blank character of each line, so this deletes the first character on each line from 1 to 10.

### Skipping Mappings with `:normal!`:

- If you have remapped keys in Normal mode but want to execute the original behavior, you can use `:normal!`.

**Example:**
- Use the unmodified `dd` (delete line) even if `dd` is remapped:
  ```vim
  :normal! dd
  ```

### Useful Scenarios:

- **Indent multiple lines:**
  ```vim
  :10,20normal >>
  ```
  This indents lines 10 to 20.

- **Jump to the last character of each line in a range:**
  ```vim
  :5,10normal $
  ```
  Moves the cursor to the end of the line on lines 5 through 10.

### Combining `:normal` with Other Commands:
You can use the `:normal` command in combination with other Ex commands like search and replace or line addressing to make complex edits.

