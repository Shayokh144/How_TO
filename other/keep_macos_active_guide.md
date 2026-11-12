# How to Keep macOS Active and Commands Running

This guide covers how to prevent your Mac from sleeping during long-running Terminal tasks, as well as how to keep commands running in the background even if you close the Terminal window.

## Part 1: Keeping macOS Awake (Using `caffeinate`)

The most efficient way to keep macOS awake during a long-running terminal task is using the built-in `caffeinate` command.

### Method 1: Wrap your command (Recommended)
Simply type `caffeinate` before the command you want to run. 

```bash
caffeinate ./my_long_script.sh
```

This prevents your Mac from sleeping for exactly as long as the script runs. Once the command finishes, your normal sleep settings instantly resume.

### Method 2: Keep the system awake independently
If you have already started your long-running command, you can open a new Terminal tab or window and run:

```bash
caffeinate -i
```

This keeps the system awake indefinitely. When your other task finishes, simply go back to this tab and press `Ctrl + C` to terminate the caffeinate process.

### Method 3: Keep the display awake too
By default, `caffeinate` lets the screen turn off while keeping the system processing in the background. If you need to monitor the terminal output visually, use the `-d` flag to keep the display on as well:

```bash
caffeinate -d ./my_long_script.sh
```

---

## Part 2: Keeping Commands Running After Closing Terminal

If you want to close the Terminal window without interrupting your task, you can use `nohup` or `tmux`. `nohup` is best for "fire and forget" background tasks, while `tmux` is ideal if you want to visually check on the command's live progress later.

### Method 1: Using `nohup` (Quick & Built-in)
`nohup` tells your system to ignore the "hangup" signal that gets sent when a terminal window closes. 

1. Run your command with `nohup` at the beginning and an ampersand `&` at the end to push it to the background:
   ```bash
   nohup ./my_long_script.sh &
   ```
2. Press `Enter` to return to your prompt. You can now safely close the Terminal window.

**Verification:** Look in the directory where you ran the command. You will see a new `nohup.out` file, which automatically captures any text or errors your script prints to the screen.

### Method 2: Using `tmux` (Robust & Interactive)
`tmux` creates a virtual terminal window that stays alive in the background. Because it doesn't come pre-installed on macOS, you may need to install it first (e.g., `brew install tmux`).

1. Start a new named session:
   ```bash
   tmux new -s my_task
   ```
2. Run your command normally inside this new terminal view:
   ```bash
   ./my_long_script.sh
   ```
3. **Detach** from the session by pressing `Ctrl + B`, letting go, and then pressing `D`. You can now safely close the Terminal window.
   *Verification:* Upon detaching, your normal terminal will reappear and print `[detached (from session my_task)]`.
4. To **reattach** and check on your command later, open a new Terminal and run:
   ```bash
   tmux attach -t my_task
   ```
   *Verification:* Your terminal will immediately snap back to your running script exactly as you left it.