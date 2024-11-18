### 1. **`C`** → **`c$`**
   - **`C`**: Change (replace) text from the cursor position to the end of the line.
   - **`c$`**: This is the longhand version of the same command, where `c` means change and `$` means until the end of the line. It replaces everything from the cursor to the end of the line with whatever you type next.

### 2. **`s`** → **`cl`**
   - **`s`**: Substitute the character under the cursor and go into insert mode.
   - **`cl`**: This is the longhand version of the same command. `c` is for change, and `l` means the current character (which stands for the next character after the cursor). It deletes the current character and puts Vim into insert mode.

### 3. **`S`** → **`^C`** (`C` here refers to "capital C")
   - **`S`**: Substitute the entire current line and go into insert mode (the entire line is deleted).
   - **`^C`** (longhand): This is the control key notation. `S` (shift + `s`) is equivalent to `^C` in this context and deletes the entire line and switches to insert mode.

### 4. **`I`** → **`^i`**
   - **`I`**: Insert at the beginning of the current line.
   - **`^i`** (longhand): This is the control key notation that also means "insert at the beginning of the current line."

### 5. **`A`** → **`$a`**
   - **`A`**: Append at the end of the current line (moves the cursor to the end of the line and enters insert mode).
   - **`$a`**: This is the longhand equivalent, where `$` moves the cursor to the end of the line, and `a` appends after the last character.

### 6. **`o`** → **`A<CR>`**
   - **`o`**: Open a new line below the current line and go into insert mode.
   - **`A<CR>`**: This is the longhand equivalent, where `A` moves the cursor to the end of the current line and `<CR>` (carriage return) means to press Enter, opening a new line below the current one.

### 7. **`O`** → **`ko`**
   - **`O`**: Open a new line above the current line and go into insert mode.
   - **`ko`**: This is the longhand equivalent, where `k` moves the cursor up one line, and `o` opens a new line below the current one (which was the original current line).
