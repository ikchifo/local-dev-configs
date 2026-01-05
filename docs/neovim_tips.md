# Neovim Tips and Tricks

## General Navigation
- `h` -> Move left
- `j` -> Move down
- `k` -> Move up
- `l` -> Move right
- `w` -> Jump forwards to the start of a word
- `b` -> Jump backwards to the start of a word
- `e` -> Jump to the end of a word
- `gg` -> Go to the start of the file
- `G` -> Go to the end of the file
- `<num>G` -> Go to line `<num>`
- `0` -> Go to the start of the line
- `$` -> Go to the end of the line
- `%` -> Jump to matching bracket
- `Ctrl + o` -> Jump to older cursor position
- `Ctrl + i` -> Jump to newer cursor position
- `^` -> Go to the first non-blank character of the line
- `<cr>` -> Carriage return / Enter key
---

## Editing Text
### Inserting and Replacing
- `a` -> Append (insert after cursor)
- `A` -> Append (insert at the end of the line)
- `i` -> Insert (insert before cursor)
- `R` -> Replace (overwrite text, Replace mode)
- `r` -> Replace a single character

### Deleting
- `x` -> Delete character under cursor
- `dd` -> Delete line
- `dw` -> Delete word
- `di"` -> Delete inside double quotes
- `di'` -> Delete inside single quotes
- `di(` -> Delete inside parentheses
- `d3w` -> Delete 3 words forwards
- `2dd` -> Delete 2 lines
- `D` or `d$` -> Delete from the cursor to the end of the line

### Changing
- `ciw` -> Change inside a word
- `cw` -> Change (replace) a word from the cursor to the end of the word
- `ce` -> Change (replace) to the end of the word
- `C` or `c$` -> Change (replace) to the end of the line

### Undo and Redo
- `u` -> Undo
- `Ctrl + r` -> Redo
- `U` -> Undo all changes on a line

---

## Copying and Pasting
- `yy` -> Yank (copy) a line
- `yw` -> Yank (copy) a word
- `yiw` -> Yank (copy) inside a word
- `y$` -> Yank (copy) to the end of the line
- `p` -> Paste after the cursor
- `P` -> Paste before the cursor
- `dd` then `p` -> Move a line down
- `dd` then `P` -> Move a line up

---

## Searching
- `/word` -> Search for `word`
- `n` -> Next occurrence
- `N` -> Previous occurrence
- `*` -> Search for the word under the cursor
- `#` -> Search backwards for the word under the cursor
- `:s/<old>/<new>/g` -> Replace `<old>` with `<new>` in the current line
- `:%s/<old>/<new>/g` -> Replace `<old>` with `<new>` in the whole file
- `:%s/<old>/<new>/gc` -> Replace `<old>` with `<new>` in the whole file with confirmation
- `:20,30s/<old>/<new>/g` -> Replace `<old>` with `<new>` from line 20 to line 30

---

## Visual Modes
- `v` -> Start visual mode (highlight text)
- `V` -> Start visual line mode (highlight whole lines)
- `Ctrl + V` -> Start visual block mode (highlight a block of text)
- `ggVG` -> Select all text in the file

---

## Working with Files
- `:r filename` -> Read the contents of `filename` into the current file
- `:r !<command>` -> Read the output of an external command into the current file
- `:vsp filename` -> Open file in vertical split
- `:tabnew filename` -> Open file in new tab
- `gt` -> Go to next tab
- `gT` -> Go to previous tab

---

## Registers and Clipboard
- `"ayw` -> Yank (copy) a word into register `a`
- `<Ctrl> + r + a` -> Paste from register `a`
- `"ap` -> Paste from register `a`
- `"+y` -> Yank to system clipboard (requires `+clipboard` feature)
- `"+p` -> Paste from system clipboard (requires `+clipboard` feature)
- `<Ctrl> + r + +` -> Paste from system clipboard (requires `+clipboard` feature)

---

## Macros
- `q<register>` -> Start recording a macro into register `<register>` (a-z)
- `q` -> Stop recording the macro
- `@<register>` -> Play the macro in register `<register>`
- `@@` -> Play the last played macro again
- `<num>@<register>` -> Play the macro in register `<register>` `<num>` times

## Marks
- `m<letter>` -> Set a mark at the current cursor position
- `"<letter>d'a` -> into register(") named (<letter>) put the (d)eletion from the cursor to the LINE containing mark(') (<letter>)
- `'<letter>` -> Go to the position of the mark `<letter>`
- `'<letter>`` -> Go to the position of the mark `<letter>` and back to the current positione

---

## Git Integration (Fugitive Vim)
### Stage Files
- Navigate to the desired file within the status window using `j` and `k` (or `Ctrl + n` and `Ctrl + p`).
- Press `-` to stage or unstage the file under the cursor.
- To stage multiple files, visually select them using `V` and then press `-`.
- To stage a specific hunk within a file:
  1. Navigate to the file.
  2. Press `=` to view the diff.
  3. Navigate to the hunk and press `-`.

### Commit Changes
- From the Git status window, with your desired changes staged:
  1. Press `cc` to initiate the commit process.
  2. A new buffer will open for you to compose your commit message.
  3. Enter your commit message, save the buffer, and quit (e.g., `:wq`) to complete the commit.

---

## Plugins and Shortcuts
### Telescope Shortcuts
- `<leader>ff` -> Find files
- `<leader>fg` -> Live grep

### Tab Shortcuts
- `<F8>` -> Go to next tab
- `<F7>` -> Go to previous tab

### LSP Autocomplete
- `Ctrl + n` / `Ctrl + p` -> Cycle forward or backward through autocomplete suggestions
- `Ctrl + y` -> Accept autocomplete suggestion

### Plugin Manager
- `:Lazy` -> Open Lazy plugin manager

---

## Miscellaneous
- `:set ic` -> Ignore case in searches
- `:set noic` -> Do not ignore case in searches
- `:set hlsearch` -> Highlight search results
- `:set nu` -> Show line numbers
- `:set invnu` -> Toggle line numbers
- `gx` -> Open the file under the cursor with the default application (e.g., URL in browser)
- `gd` -> Go to the definition of a variable or function under the cursor (if supported by LSP)
- `:abbr pgn penguin` -> Create an abbreviation (e.g., typing `pgn` will replace it with `penguin`)
- `:unab pgn` -> Remove the abbreviation for `pgn`
- `va)` -> Visually Select around a `[)]` parentheses
- `vi)` -> Select inside a paragraph
- `ci'` -> Change inside quotes
- `:bp` -> Go to the previous buffer
- `:bn` -> Go to the next buffer
- `:ls` -> List all buffers

- In insert mode, `<CTRL-r>=60*60 <ENTER>` -> Insert the result of the expression `60*60`
