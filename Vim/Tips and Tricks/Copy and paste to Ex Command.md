In Vim, to copy and paste text directly into the `:` (ex) command line, you can follow these steps:

1. **Copying Text in Vim:**
   - In normal mode, visually select the text you want to copy by pressing `v` (character-wise) or `V` (line-wise).
   - Press `y` to yank (copy) the selected text.

2. **Pasting into the `:` Command Line:**
   - Type `:`, which opens the command-line mode.
   - Press `Ctrl` + `r` followed by `"` to paste the last yanked text into the command line.

This method works because `Ctrl` + `r` in the command-line mode lets you insert the contents of registers, and `"` is the default register for the last yanked or deleted text.