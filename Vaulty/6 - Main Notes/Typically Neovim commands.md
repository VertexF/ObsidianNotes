2026-07-01 16:49
Status: #baby 
Tags: [[linux development]]
# Typically Neovim commands

If you want to open nvim you need to type `nvim` if you want to open file in nvim you type `:edit path/to/file`


This allows you to go through a source to select. `:Exp` 

You can undo with `u` and redo with `ctrl-r` all in command mode.

If you use `/` you can search for a string in a file, if you press enter your cursor close to that location. `n` and `p` go from next to previous.

If you end up in mode you don't like you can always quit with `:q!`

You have buffers with nvim, when you type `:%` you are using the visual buffer aka everything the screen. Meaning that you can do a search and replace with `:%s/oldName/newName`  If you want every instance of the word to change and not just first in a sequence of words you need to add`g` like this `:%s/oldName/newName/g`.

If you want to run any bash command you can use `:!` and then any regular command. 

If you want to bring up a terminal you can simply add `:vert term` or `:hori term` and it will pop up. Once you have finished you can just `:q` to get back to your text.

If you want to select more than 1 line you press `v` to go into visual mode, then you press `ctrl+v` to select more than 1 line by pressing the arrow keys. Then you can delete more than 1 line at a time.

movement:
w - word
W - WORD (includes spaces and stuff)
y - yank (copy)
d - cut
c - cut and enter insert mode
^ double does the line u're currently on
p - paste
P - paste below

gg - start of file
G - end of file
<number>G - go to line

$ - end of line
g_ - end of line before the newline character
^ - start of line (useful if u have tabs and wanna go to the start of where characters is)
0 - start of line

V - visual line mode
v - visual mode

A - append to end of line
a - append
i - insert (append but it's before ur character)
I - insert at the start of line
o - insert below line
O - insert above line

useful commands (u don't need colon for this):
t<char> - till
f<char> - find character
movement analogs similar to append/insert. till goes before character, find goes ontop the character.

ci - change inside, useful if u need to remove stuff inside a () in front
di - delete inside
yi - yank inside

ca - change around, this just includes the () themselves
same da, ya etc

if it says O-PENDING it means operator pending 

##### Indenting 
@TT-hi7lp​​>i{ to indent inside { bracket

@TT-hi7lp​​>- de-indent

@TT-hi7lp​​>a{ for indent (a)round { bracket meaning also the line with the {

# References
##### Main Notes
[[Typically tmux commands]]
#### Source Notes
