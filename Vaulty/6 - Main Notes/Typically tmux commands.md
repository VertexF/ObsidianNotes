2026-07-01 19:12
Status: #baby 
Tags: [[Linux Development]]
# Typically tmux commands

If you want start a tmux session you need to you just start `tmux`

To send tmux session you need to type in a `tmux kill-session` within a tmux session.

To split the window you type vertically  `ctrl + b` follows by `%` horizontal is the same but `ctrl + b` follows by `"` 

To switch panels you just go `ctrl + b` follows by the arrow keys `->` `<-` `^` `v` If you want to switch between your last active window you can type `ctrl + b` follows by `;` 

Exiting the actual window keeps the tmux session alive with maybe whatever you wanted in it. So you need to kill the session if you truly want it closes.

This starts an ssh-agent to add ssh keys to things.
```shell
eval "$(ssh-agent -s)"
```
# References
##### Main Notes
[[Typically Neovim commands]]
#### Source Notes
