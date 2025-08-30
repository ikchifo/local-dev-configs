
## Neovim
[space] + S + H = Search help

Many commands are a combination of 1) operator (what to do) and 2) motion (where to do it).
h -> left
j -> down
k -> up
l -> right

a -> append (insert after cursor)
A -> append (insert at end of line)s
i -> insert (insert before cursor)
R -> replace (overwrite text)(Replace mode)
x -> delete character under cursor
dd -> delete line
dw -> delete word
u -> undo
. -> repeat last command
Ctrl + r -> redo

w -> jump forwards to the start of a word
b -> jump backwards to the start of a word
e -> jump to the end of a word

gg -> go to the start of the file
G -> go to the end of the file

yy -> yank (copy) a line
yw -> yank (copy) a word
y$ -> yank (copy) to the end of the line
p -> paste after the cursor
P -> paste before the cursor

0 -> go to the start of the line
\$ -> go to the end of the line

O -> open a new line above the current line
o -> open a new line below the current line

cw -> change (replace) a word
ce -> change (replace) to the end of the word
C or c$ -> change (replace) to the end of the line

D or d$ -> delete from the cursor to the end of the line
r -> replace a single character

<num>w -> jump forwards <num> words

U -> undo all changes on a line

dd then p -> move a line down
dd then P -> move a line up

d3w -> delete 3 words forwards
2dd -> delete 2 lines

<num>G -> go to line <num>
/<word> -> search for <word> && n -> next occurrence && N -> previous occurrence

Ctrl + o -> jump to older cursor position
Ctrl + i -> jump to newer cursor position

% -> jump to matching bracket

:s/<old>/<new>/g -> replace <old> with <new> in the current line
:%s/<old>/<new>/g -> replace <old> with <new> in the whole file
:%s/<old>/<new>/gc -> replace <old> with <new> in the whole file with confirmation
:20,30s/<old>/<new>/g -> replace <old> with <new> from line 20 to line 30

* -> search for the word under the cursor
'#' -> search backwards for the word under the cursor

!<command> -> run an external command

v -> start visual mode (highlight text) (then you can use y, d, c, etc.)
V -> start visual line mode (highlight whole lines)
Ctrl + V -> start visual block mode (highlight a block of text) then Insert insert text in multiple lines + Esc

:r filename -> read the contents of filename into the current file
:r !<command> -> read the output of an external command into the current file

:set ic -> ignore case in searches
:set noic -> do not ignore case in searches
:set hlsearch -> highlight search results
:set nu -> show line numbers
:set invnu -> toggle line numbers

Ctrl + w -> jump to the next window
:<first letter of command> + (Ctrl + d) -> show all commands starting with <first letter of command>

gx -> open the file under the cursor with the default application (if it's a URL, it opens in the browser)
gd -> go to definition of a variable or function under the cursor (if supported by LSP)

:abbr pgn penguin -> create an abbreviation (when you type 'pgn' and then a non-keyword character, it will be replaced with 'penguin')
:unab pgn -> remove the abbreviation for 'pgn'
