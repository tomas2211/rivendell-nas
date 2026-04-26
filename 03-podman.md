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

# commands to simplify using systemctl
alias scstat="systemctl --user status"
alias scstrt="systemctl --user daemon-reload && systemctl --user start"
alias screstrt="systemctl --user daemon-reload && systemctl --user restart"
alias scstp="systemctl --user stop"
alias jc="journalctl --user -fxeu"

# I went a bit fancy with colors of the prompt
PS1='${debian_chroot:+($debian_chroot)}\[\e[91;1m\]\u\[\e[32m\]@\[\e[32m\]\h\[\e[32m\]:\[\e[34m\]\w\[\e[0m\]\\$ '
```
