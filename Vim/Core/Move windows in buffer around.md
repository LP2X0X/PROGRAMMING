Ah, got it! If you're looking to **move windows (splits) around** in **GVim**, like rearranging vertical and horizontal splits, here's what you can do:

---

## 🪟 1. **Manually Move Windows Using `Ctrl-w` Commands**

Vim provides commands to rearrange windows (splits):

|Action|Shortcut|
|---|---|
|Move current window **left**|`:wincmd H` or `Ctrl-w H`|
|Move current window **right**|`:wincmd L` or `Ctrl-w L`|
|Move current window **up**|`:wincmd K` or `Ctrl-w K`|
|Move current window **down**|`:wincmd J` or `Ctrl-w J`|

> These commands **move the _entire window_** (along with the buffer it displays) to a different position in the layout.

### 💡 Example:

If you have a vertical split and you want the current window on the left:

```vim
Ctrl-w H
```

If you want it on the bottom:

```vim
Ctrl-w J
```

---

## 🗂 2. **Swap Contents Between Windows**

If you want to **swap the contents (buffers)** of two windows rather than moving the layout:

1. Go to Window A:
    
    ```vim
    :let bufA = bufnr('%')
    ```
    
2. Go to Window B:
    
    ```vim
    :let bufB = bufnr('%') | exec 'buffer ' . bufA
    ```
    
3. Return to Window A:
    
    ```vim
    :exec 'buffer ' . bufB
    ```
    

---

## 🧠 Bonus: Use Mappings (Optional)

To make window moving faster, add to your `.vimrc` or use `:map` in GVim:

```vim
nnoremap <Leader>wh <C-w>H
nnoremap <Leader>wl <C-w>L
nnoremap <Leader>wj <C-w>J
nnoremap <Leader>wk <C-w>K
```

Now `<Leader>wh` moves the window to the left, etc.
