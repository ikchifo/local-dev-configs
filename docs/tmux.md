# TMUX Tips and Tricks


- Install Tmux plugin manager:
  ```
  git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
  ```

- `tmux -name session_name` -> Start a new tmux session with the name `session_name`
- `tmux attach -t session_name` -> Attach to an existing tmux session named `session_name`
- `tmux ls` -> List all tmux sessions
- `tmux kill-session -t session_name` -> Kill the tmux session named `session_name`

- `Ctrl + Space` -> Prefix key (changed from `Ctrl + b` in `~/.tmux.conf`)

- `prefix + :new` -> Create a new session
- `prefix + s` -> List all sessions
- `prefix + d` -> Detach from the current session

- `prefix + c` -> Create a new window
- `prefix + ,` -> Rename the current window
- `prefix + w` -> List all windows
- `prefix + n` -> Go to the next window
- `prefix + p` -> Go to the previous window 

- `prefix + &` -> Close the current window
- `prefix + x` -> Close the current pane

- `prefix + %` -> Split the current pane vertically
- `prefix + "` -> Split the current pane horizontally
- `prefix + arrow keys` -> Navigate between panes
- `prefix + q` -> Show pane numbers for quick navigation
- `prefix + z` -> Toggle fullscreen for the current pane
