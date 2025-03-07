set clipboard^=unnamed " Set clipboard = unnamedplus " To always use clipboard for all operation
set number " For number of lines
set backspace=indent,eol,start " For using backspace
set nocompatible " Turns off legacy vi mode
set encoding=utf-8
set nospell " No spellcheck
set nohls " No highlight search
set smartcase
set ic
let @/ = ""
let skip_loading_mswin=1

" Set the font and font size 
if has('gui_win32') || has('gui_win64') || has('gui_running') || has('gui_gtk2')
  silent! call simple_guifont#Set(
    \[], 'Cascadia Mono', 13)
endif

filetype off " Required!

" Set the runtime path to include Vundle and initialize
set rtp+=C:\Users\admin\~\.vim\bundle\Vundle.vim
call vundle#begin() " Begin of plugins list

Plugin 'lambdalisue/vim-fullscreen'
Plugin 'gmarik/Vundle.vim'
Plugin 'morhetz/gruvbox'
Plugin 'vim-scripts/indentpython.vim'
Plugin 'scrooloose/nerdtree'
Plugin 'dense-analysis/ale'
Plugin 'vim-syntastic/syntastic'
Plugin 'nvie/vim-flake8'
Plugin 'junegunn/fzf'
Plugin 'junegunn/fzf.vim'
Plugin 'itchyny/lightline.vim'
Plugin 'artanikin/vim-synthwave84'
Plugin 'awvalenti/vim-simple-guifont'
Plugin 'rust-lang/rust.vim'

call vundle#end() " End of plugin list
filetype plugin indent on     " Required!

let g:lightline = {
      \ 'colorscheme': 'one',
      \ 'active': {
      \   'left': [ [ 'mode', 'paste' ],
      \             [ 'readonly', 'filename', 'modified', 'author' ] ]
      \ },
      \ 'component': {
      \   'author': 'LP2X0X'
      \ },
      \ }

set laststatus=2 "Always on Lightline

autocmd vimenter * ++nested colorscheme gruvbox " Set the color scheme
set bg=dark

set undofile " For undo after reboot

au BufNewFile,BufRead *.py set tabstop=4|set softtabstop=4|set shiftwidth=4|set textwidth=0 |set expandtab|set autoindent|set fileformat=unix

highlight  BadWhitespace ctermbg=red guibg=red

au BufRead,BufNewFile *.py,*.pyw,*.c,*.h match BadWhitespace /\s\+$/

let python_highlight_all=1
syntax on

let g:ale_linters = {
			\ 'python': ['flake8', 'pylint'],
			\ 'ruby': ['standardrb', 'rubocop'],
			\ 'javascript': ['eslint'],
			\}
let g:ale_fixers = {
			\    'python': ['yapf'],
			\}

set foldenable
set foldmethod=indent
set foldlevel=99


au VimEnter * NERDTree

set complete+=kspell
set completeopt=menuone,longest
set spelllang=en


" HOTKEYS "
" Spell check
map <F1> :setlocal spell!<CR> 
" Map NERDTree to F2
map <F2> :NERDTreeToggle<CR> 
" Full screen mode toggle
map <F3> <C-Enter> :source $MYVIMRC<CR>
" Highlight search toggle
noremap <F4> :set hlsearch! hlsearch?<CR>
" Run Python code
imap <F5> <Esc>:w<CR>:!python %<CR>
nmap <F5> <Esc>:w<CR>:!python %<CR>
" Remap arrow keys to resize window
nnoremap <Up>    :resize -1<CR>
nnoremap <Down>  :resize +1<CR>
nnoremap <Left>  :vertical resize -1<CR>
nnoremap <Right> :vertical resize +1<CR>
" Toggle foldmethod to indent
map <F6> :set foldmethod=indent<CR>
" Toggle foldmethod to manual
map <F8> :set foldmethod=manual<CR>
" Map select all to Ctrl + A in normal mode
"nnoremap <C-a> ggVG
" Paste to Vim cmd (in command mode)
cnoremap <C-v> <C-r>*
" Open vimrc to edit
map <F7> :e $MYVIMRC<CR>
" Map jk to <ESC>
inoremap jk <Esc>



" Configure for spell check
hi SpellBad cterm=underline "ctermfg=203 guifg=#ff5f5f 
hi SpellLocal cterm=underline "ctermfg=203 guifg=#ff5f5f 
hi SpellRare cterm=underline "ctermfg=203 guifg=#ff5f5f 
hi SpellCap cterm=underline "ctermfg=203 guifg=#ff5f5f


set shortmess+=A "For swap file pop-up error
set hlsearch
set wildmenu	
set incsearch
set shortmess-=S "Show match count

" Treat all numerals as decimal
set nrformats=

set tabstop=4
set softtabstop=4
set shiftwidth=4

set scrolloff=5
set history=700

" SYNTASTIC Configure
set statusline+=%#warningmsg#
set statusline+=%{SyntasticStatuslineFlag()}
set statusline+=%*

let g:syntastic_always_populate_loc_list = 1
let g:syntastic_auto_loc_list = 1
let g:syntastic_check_on_open = 1
let g:syntastic_check_on_wq = 0

