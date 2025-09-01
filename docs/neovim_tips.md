
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
Once in search mode:
   + n -> next occurrence
   + N -> previous occurrence
   + Enter -> repeat last search
   or c + i + w -> change the word under the cursor
      then Esc + n -> go to next occurrence + . -> repeat the change

!<command> -> run an external command

v -> start visual mode (highlight text) (then you can use y, d, c, etc.)
   + "}" or "{" -> go to next or previous empty line
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

:vsp filename -> open file in vertical split
:tabnew filename -> open file in new tab
gt -> go to next tab
gT -> go to previous tab

va)  - [ V ]isually select [ A ]round [ ) ]paren

Gdiffsplit -> open a vertical split showing git diffs (needs fugitive.vim plugin)


double quotes + <num> + p -> paste from register <num> (0-9)

double quotes + plus + y -> yank to system clipboard (requires +clipboard feature)
double quotes + plus + p -> paste from system clipboard (requires +clipboard feature)


### LSP Autocomplete

Ctrl + n / Ctrl + p --> Cycle forward or backward through autocomplete suggestions
Ctrl + y --> Accept autocomplete suggestion

:Lazy -> open Lazy plugin manager

### Shortcuts

- nnoremap <leader>ff <cmd>lua require('telescope.builtin').find_files()<cr>
- nnoremap <leader>fg <cmd>lua require('telescope.builtin').live_grep()<cr>
- nnoremap <F8> gt
- nnoremap <F7> gT


### Macros
q + <register letter> -> start recording a macro into register <register letter> (a-z)
q -> stop recording the macro
@ + <register letter> -> play the macro in register <register letter>
@@ -> play the last played macro again
<num> + @ + <register letter> -> play the macro in register <register letter> <num> times


### Fugitive Vim (Git integration)

#### Stage Files:
- Navigate to the desired file within the status window using j and k (or C-n and C-p).
- Press - (dash) to stage or unstage the file under the cursor.
- To stage multiple files, visually select them using V and then press -.
- To stage a specific hunk (a block of changes) within a file, navigate to the file, press = to view the diff, then navigate to the hunk and press -.

#### Commit Changes:
- From the Git status window, with your desired changes staged, press cc to initiate the commit process.
- A new buffer will open for you to compose your commit message.
- Enter your commit message, save the buffer, and quit (e.g., :wq) to complete the commit.
