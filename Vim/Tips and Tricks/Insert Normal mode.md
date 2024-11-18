- In Vim, you can run a single Normal mode command while in Insert mode and automatically return to Insert mode using **`<C-o>`** (Control + `o`). This allows you to execute a single Normal mode command without fully leaving Insert mode.

### How to Use `<C-o>`:
1. **In Insert mode**, press `Ctrl + o`.
2. **Type your Normal mode command** (e.g., `dw` to delete a word, `u` to undo, etc.).
3. After the command runs, Vim will automatically return to Insert mode.

### Example Usage:
- If you are in Insert mode and want to delete a word before the cursor:
  - Press `Ctrl + o`, type `db`, and Vim will delete the word and return to Insert mode.

- If you want to move to the beginning of the line and continue typing:
  - Press `Ctrl + o`, type `0`, and you're moved to the start of the line. Insert mode resumes right after.

This is very handy for quick edits without fully switching between modes.