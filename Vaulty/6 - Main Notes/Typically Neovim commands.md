2026-07-01 16:49
Status: #baby 
Tags: [[Linux Development]]
# Typically Neovim commands

This allows you to go through a source to select. `:Exp` 

You can undo with `u` and redo with `ctrl-r` all in command mode.

If you use `/` you can search for a string in a file, if you press enter your cursor close to that location. `n` and `p` go from next to previous.

If you end up in mode you don't like you can always quit with `:q!`

You have buffers with nvim, when you type `:%` you are using the visual buffer aka everything the screen. Meaning that you can do a search and replace with `:%s/oldName/newName`  If you want every instance of the word to change and not just first in a sequence of words you need to add`g` like this `:%s/oldName/newName/g`.

If you want to run any bash command you can use `:!` and then any regular command. 

If you want to bring up a terminal you can simply add `:vert term` or `:hori term` and it will pop up. Once you have finished you can just `:q` to get back to your text.

# References
##### Main Notes
[[Typically tmux commands]]
#### Source Notes
