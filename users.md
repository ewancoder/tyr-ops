# Users

> TODO: Consider doing the same for DO servers.

I have a special `github` user created on my machine, with limited permission and rootless Docker access, for deployments from github actions.

> Better name would have been `tyr`, consider renaming this user later.

`2000` - is the user ID. Set up all project folder access respective to this user.

- `sudo useradd -u 2000 -m -s /bin/bash github`, `-m` creates home directory
- `sudo passwd -l github` - locks the account so it can't use password
- `sudo -iu github` - simulates login with this user
- `curl -fsSL https://get.docker.com/rootless | sh` - installs local user-scoped copy of rootless docker
- Add variables from printed instructions to ~/.bashrc (for SSH sessions)
  - `export PATH=$HOME/bin:$PATH`
  - `export XDG_RUNTIME_DIR=$HOME/.docker/run`
  - `export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock`
- `source ~/.bashrc`
- `dockerd-rootless.sh` (to check)
- `loginctl enable-linger github` - allow services to run even with user logged out
  - `loginctl show-user github` - check Linger property

Put the following into `~/.config/systemd/user/docker.service`:

```
[Unit]
Description=Rootless Docker Daemon
After=network.target

[Service]
ExecStart=/home/github/bin/dockerd-rootless.sh
Restart=always
Environment=PATH=/home/github/bin:/usr/bin:/bin
Environment=XDG_RUNTIME_DIR=/home/github/.docker/run
Environment=DOCKER_HOST=unix:///home/github/.docker/run/docker.sock

[Install]
WantedBy=default.target
```

- `export XDG_RUNTIME_DIR=/run/user/$(id -u)` - starts a temporary user-scoped systemd session so we don't need to reboot to continue setting it up
- `systemctl --user daemon-reload`
- `systemctl --user enable docker.service`
- `systemctl --user start docker.service`
- `docker info` - should show **Rootless: true**
- `reboot`

Additionally, add the needed public key to `~/.ssh/authorized_keys` for SSH connection.

## Very important

We need to add the docker that is a rootless 'github' user-scoped instance as a Swarm NODE in order for it to see the networks. Otherwise even the straightforward deployment fails.
