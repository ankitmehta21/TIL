Adding the following snippet to your .vimrc will insert a datetime into a
newline below the cursor when you strike <leader> d (`,d` by default).
Very handy for logging notes:

```
nnoremap <leader>d :put =strftime('%Y-%m-%d %H:%M')<cr>
```
