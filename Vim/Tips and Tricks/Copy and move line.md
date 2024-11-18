In Vim's ex command mode (command-line mode), you can use the `:copy` and `:move` commands to duplicate or move lines.

### 1. **Copy Lines**
- The `:copy` command (or `:t` as its shorthand) copies lines from one place to another.
- Syntax: `:range copy {address}`
  
  **Example:**
  - `:3copy 5` or `:3t 5` copies line 3 to line 5.
  - `:2,5copy 10` copies lines 2 through 5 after line 10.

### 2. **Move Lines**
- The `:move` command moves lines from one place to another.
- Syntax: `:range move {address}`

  **Example:**
  - `:3move 5` moves line 3 to after line 5.
  - `:2,5move 10` moves lines 2 through 5 to after line 10.

### Range Examples:
- `:.,$copy 10` copies the current line through the last line to after line 10.
- `:%move 1` moves all lines in the file to after line 1.

### Addressing:
- `.` refers to the current line.
- `$` refers to the last line.
- `%` refers to the entire file.
