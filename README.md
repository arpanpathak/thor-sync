# thor-sync

Keep folders on your laptop or desktop copied to a Jetson (or any Linux box
you can SSH into), so you edit locally and build or run on the device.

```bash
thor-sync add          # in a project folder: put it on the list and sync it
thor-sync on           # from now on, every change reaches the device in seconds
```

One bash script. It needs `bash`, `rsync` and `ssh` on both machines, nothing
else.

## Install

```bash
git clone https://github.com/arpanpathak/thor-sync
ln -s "$PWD/thor-sync/thor-sync" ~/.local/bin/thor-sync
```

## Connect to the device

thor-sync talks to the device over SSH without a password. Pick one.

### On the same network

```bash
ssh-keygen -t ed25519 -N "" -f ~/.ssh/id_ed25519     # skip if you have a key
ssh-copy-id user@192.168.0.83                        # asks for the password once
thor-sync host user@192.168.0.83
```

### From anywhere, with Tailscale

[Tailscale](https://tailscale.com) gives each machine a stable name that works
from any network, and Tailscale SSH replaces SSH keys with your Tailscale login.

On the device:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --ssh
```

On your machine (install Tailscale the same way, then `sudo tailscale up`):

```bash
tailscale status                      # find the device's name, e.g. thor
ssh user@thor true                    # must work without a password
thor-sync host user@thor
```

### With an SSH alias

Any `Host` from `~/.ssh/config` works too, e.g. `thor-sync host thor`. Adding
`ControlMaster auto`, `ControlPath ~/.ssh/cm-%r@%h:%p` and `ControlPersist 10m`
to that block makes every sync start faster.

## Use

| Command | What it does |
|---|---|
| `thor-sync` | sync every folder on the list |
| `thor-sync add [DIR]` | add a folder (default: the current one) and sync it |
| `thor-sync rm [DIR]` | take a folder off the list; its copy on the device stays |
| `thor-sync ls` | show the list |
| `thor-sync watch` | sync every 2 seconds in this terminal |
| `thor-sync on` / `off` | background sync as a systemd user service, kept across reboots |
| `thor-sync log` | follow the background sync |
| `thor-sync host [NAME]` | show or set the device |

## What it copies

- A folder under your home lands at the same place under the device's home:
  `~/Projects/app` becomes `~/Projects/app` there.
- Git history is copied, so `git` works on the device.
- Skipped: whatever each folder's `.gitignore` skips, plus the patterns in
  `~/.config/thor-sync/exclude` (build output such as `target/` and
  `node_modules/`, virtualenvs, model weights such as `*.gguf` and
  `*.safetensors`). Edit that file to change it.
- One way only, local to device. Deleting a file locally never deletes it on
  the device, so output produced there is safe. Commit locally: a commit made
  on the device is overwritten by the next sync.

## Files

| Path | Holds |
|---|---|
| `~/.config/thor-sync/folders` | the list, one absolute path per line |
| `~/.config/thor-sync/exclude` | rsync exclude patterns |
| `~/.config/thor-sync/host` | the device |
| `~/.config/systemd/user/thor-sync.service` | the background service, made by `thor-sync on` |
