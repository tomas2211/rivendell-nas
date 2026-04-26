# Podman

Let's setup podman and tooling around it.

## Install

Installing through apt gets you a bit older version. Alternative source would be brew, but that didn't seem to work in multi-user setup. At the end, I preffered the multi-user setup.

1. `sudo apt install podman`
 - You can also install cockpit for some web UI: `sudo apt install cockpit cockpit-podman` (although the usability for non-sudo users is limited).

## Prepare for adding new users

I'm adding a new user for each group of services. To prepare for this, we'll slighltly modify the contents of `/etc/skel` which a default home folder that gets copied when a new user is created.

1. Disable reading by other users `sudo chmod o-rwx /etc/skel`
2. Add/modify `/etc/skel/.bashrc`:
```bash
# basics
alias ll='ls -lah'
alias la='ls -Ah'
alias l='ls -CF'

# systemd bus - these variables are not set when su <user>
export XDG_RUNTIME_DIR=/run/user/$(id -u)
export DBUS_SESSION_BUS_ADDRESS=unix:path=${XDG_RUNTIME_DIR}/bus

# systemd --user aliases

# Daemon
alias scd='systemctl --user daemon-reload && echo "User daemon reloaded."'

# Status / listing
alias scst='systemctl --user status'
alias sclu='systemctl --user list-units --type=service'
alias scluf='systemctl --user list-unit-files'
alias sclt='systemctl --user list-timers --all'

# Start / stop / restart
alias scstart='systemctl --user start'
alias scstop='systemctl --user stop'
alias scr='systemctl --user restart'
alias screl='systemctl --user reload'

# Enable / disable
alias sce='systemctl --user enable'
alias scdi='systemctl --user disable'
alias scen='systemctl --user enable --now'
alias scdn='systemctl --user disable --now'

# Logs (journalctl)
alias jl='journalctl --user -u'
alias jlf='journalctl --user -u -f'        # follow/live
alias jlb='journalctl --user -b -u'        # since last boot
alias jle='journalctl --user -u -p err'    # errors only

# Combined convenience
alias scenl='f() { systemctl --user enable --now "$1" && journalctl --user -u "$1" -f; }; f'
alias scrl='f() { systemctl --user restart "$1" && journalctl --user -u "$1" -f; }; f'

# I went a bit fancy with colors of the prompt
PS1='${debian_chroot:+($debian_chroot)}\[\e[91;1m\]\u\[\e[32m\]@\[\e[32m\]\h\[\e[32m\]:\[\e[34m\]\w\[\e[0m\]\\$ '
```
