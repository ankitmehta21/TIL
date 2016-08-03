If `:echo has('clipboard')` returns 0 in vim, then your version of vim cannot write to the system clipboard.
Install vim-gtk instead. 
Then `"+yy` will yank to the system clipboard instead.
