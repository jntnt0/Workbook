# Linux Background Process Management

## Overview

Reference for managing foreground and background processes in a Linux terminal session. Covers suspending, backgrounding, foregrounding, and keeping processes alive after logout.

---

## Sending a Running Process to the Background

**Step 1 — Suspend the process:**

Press `Ctrl+Z` while the process is running in the foreground. The terminal will output something like:

```
[1]+  Stopped    <command>
```

**Step 2 — Resume it in the background:**

```bash
bg %1
```

The `%1` refers to the job number. Use `jobs` to list all active jobs and their numbers.

---

## Listing Background Jobs

```bash
jobs
```

Output example:

```
[1]+  Running    <command> &
[2]-  Stopped    <another-command>
```

---

## Bringing a Process Back to the Foreground

```bash
fg %1
```

Replace `%1` with the job number of the process you want to bring forward.

---

## Detaching a Process from the Terminal

Use `disown` to fully detach a background job so it continues running even if the terminal is closed:

```bash
disown %1
```

After `disown`, the process is no longer tied to the terminal session and will survive logout.

---

## Persistent Background Processes

For long-running processes that need to survive terminal closure or logout, use one of the following:

### nohup

Prepend `nohup` when starting a command to make it immune to hangup signals:

```bash
nohup <command> &
```

Output is redirected to `nohup.out` in the current directory by default. Redirect explicitly if needed:

```bash
nohup <command> > /var/log/<output>.log 2>&1 &
```

### tmux

Create a persistent session that can be detached and reattached:

```bash
# Start a new session
tmux

# Run your command inside tmux
<command>

# Detach from the session (process keeps running)
Ctrl+b, then d

# Reattach later
tmux attach-session
```

List existing sessions:

```bash
tmux ls
```

Attach to a specific session:

```bash
tmux attach-session -t <session-name>
```

---

## Quick Reference

|Action|Command|
|---|---|
|Suspend foreground process|`Ctrl+Z`|
|Resume suspended job in background|`bg %<job>`|
|Bring background job to foreground|`fg %<job>`|
|List all background jobs|`jobs`|
|Detach job from terminal|`disown %<job>`|
|Start command immune to hangup|`nohup <command> &`|
|Start persistent terminal session|`tmux`|
|Detach from tmux session|`Ctrl+b`, then `d`|
|Reattach to tmux session|`tmux attach-session`|