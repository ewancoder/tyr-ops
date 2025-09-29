# Docker

I'm using Docker for all my pet projects deployments, so this article will mostly contain my specific TyR setup.

## User setup

To allow user to use docker without the sudo command, add it to the `docker` group:

- `sudo usermod -aG docker myuser`
- `newgrp docker` - applies the group changes without needing to sign out

## TyR user setup

- I have a `tyr` user for deployments, id 2000, that's in the `docker` group.
    - `sudo useradd -u 2000 -m -s /bin/bash tyr`
    - `sudo passwd -l tyr`
    - `sudo usermod -aG docker tyr`
- Additionally, add the needed public key to `~/.ssh/authorized_keys` for SSH connection.
- All project files are 600/700 and chown-ed by this user (unless created by the container).

## Swarm

Useful commands:

- `docker node ls` - list nodes
- `docker node inspect ALIAS/ID` - show node info (for example, current labels)

Swarm nodes can have multiple labels. To add a label:

- `docker node update --label-add labelname=labelvalue ALIAS/ID`

Since nodes are not tags, we cannot set multiple values for the same key. So we need some kind of workaround to have a flexible configuration. My setup is the following:

### My nodes

- **do-main** - Digital Ocean main (manager) node
  - Label: infra=true
  - Label: worker=true
  - Label: id=domain - this is used when we need to deploy something 100% to a SINGLE main node
- **do-worker** - Digital Ocean worker node
  - Label: worker=true
- **ivanpc** - My current PC
  - Label: dev-infra=true
  - Label: dev-worker=true

When deploying services:

- Infrastructure should be only deployed to `infra=true` nodes. If we have more than one infra node - we need to set up replication (for databases / cache / etc). Currently only the manager node has infra on it.
- Stateless services should only be deployed to `worker=true` nodes. This way we distribute the load between multiple droplets.
- Development environment is deployed ONLY to `dev-infra/worker=true` nodes following the same rules.

Scenario usages:

- We want to set up infrastructure clustering: we add more droplets with `infra=true` and set up infrastructure on all the nodes.
- We want to spread the load / increase total RAM capacity. We add more `worker=true` nodes and rebalance the Swarm.
- We want to have development environment(s) running across multiple PCs. We add more PCs as dev-infra/worker nodes.

### Nodes data

Due to distributed nature of Swarm cluster, and the lack of centralized configuration server (yet), we need to make sure all necessary data is copied to all the nodes before deployments.

The following should always be present on every node:

1. DataProtection certificate (centralized across services)
2. Data folder for the service (`aircaptain_prod`, for prod env and aircaptain app; `aircaptain_dev` etc)
  - DataProtection folder for per-service keys (empty, for volume)
  - secrets.env / secrets-compose.env files with per-service secrets

For example, **any** node that has `infra` or `worker` label, should have folders for **all** non-development environments. **Any** node that has `dev-infra` or `dev-worker` label, should have folders for **all** development environments.

At this time, we only have one of each: `prod` (production) and `dev` (development).

For development environment, we store the data of a project on the PC at: `/mnt/data/pet/project`, this folder is symlinked to `/data/project`.

> When copying configs between servers while moving between environments - make sure to actually edit `secrets.env` files for the specific environment, not to allow for example `dev` environment to write into `prod` database.

> When deploying **Development** environment in **Swarm** configuration - we should also have the **secrets.env** / data folder in the `/data/pet/projectname` location **on the Swarm MASTER node**, as it is the **Swarm MASTER** that performs the deployment and needs these secrets. This is an important note that should be incorporated into the TyR deployment flow. Due to the nature of development environmens - we only need secrets on the master node, nothing else. Everything else runs on the personal PC (but personal PC probably also needs secrets).

> Due to the fact that we need to store a bunch of duplicated secrets on all the nodes - it is very relevant to migrate to some kind of centralized key/vault solution as soon as possible! Until this is done - do not forget to update the secrets in ALL the relevant places / nodes.

## Rootless Docker setup

> WARNING: this setup will not work with Swarm, Swarm requires root privileges. This is for running a separate Docker instance in a rootless mode.

This example creates a user with ID 2000, and name "github" that has a personal Rootless Docker instance.

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

## Networking

### Setting up aliases for a specific service

Let's assume you deploy a stack with a service `my-seq` from folder `my-stack-name`, so now you have a service `my-stack-name_my-seq`. But you want other services to talk to it by the `seq` name. This can be achieved with the use of aliases within a network.

When configuring networks, set them up like this:

```
networks:
  some_other_network: {}
  my_network:
    aliases:
      - seq
```

Now your service will be accessible by the domain name `seq` within the `my_network` network.

> If some other service is using this alias - stack will not deploy.
