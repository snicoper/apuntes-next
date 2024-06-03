---
sidebar_position: 1
title: "vimrc"
id: vimrc
---

```bash
set pastetoggle=<F2>
set clipboard=unnamed
set encoding=utf8

" Rebind <Leader> key
let mapleader = ","

" Enable syntax highlighting
" You need to reload this file for the change to apply
filetype off
filetype plugin indent on
syntax on
set scrolloff=8

" Showing line length
set nonu
set tw=79   " width of document (used by gd)
set nowrap  " don't automatically wrap on load
set fo-=t   " don't automatically wrap text when typing
set colorcolumn=100
highlight ColorColumn ctermbg=233

" Useful settings
set history=700
set undolevels=700

" Real programmers don't use TABs but spaces
autocmd Filetype html setlocal expandtab tabstop=2 shiftwidth=2
autocmd Filetype js setlocal expandtab tabstop=2 shiftwidth=2
autocmd Filetype scss setlocal expandtab tabstop=2 shiftwidth=2
set tabstop=4
set softtabstop=4
set shiftwidth=4
set shiftround
set expandtab

" Make search case insensitive
set hlsearch
set incsearch
set ignorecase
set smartcase

" Disable stupid backup and swap files - they trigger too many events
" for file system watchers
set nobackup
set nowritebackup
set noswapfile

" Split navigations
nnoremap <C-J> <C-W><C-J>
nnoremap <C-K> <C-W><C-K>
nnoremap <C-L> <C-W><C-L>
nnoremap <C-H> <C-W><C-H>

" Stupid shift key fixes
command! -bang -nargs=* -complete=file E e<bang> <args>
command! -bang -nargs=* -complete=file W w<bang> <args>
command! -bang -nargs=* -complete=file Wq wq<bang> <args>
command! -bang -nargs=* -complete=file WQ wq<bang> <args>
command! -bang Wa wa<bang>
command! -bang WA wa<bang>
command! -bang Q q<bang>
command! -bang QA qa<bang>
command! -bang Qa qa<bang>
```
