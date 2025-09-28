# Droplet setup

This is a complete sequence of all the things I'm doing to set up a freshly added worker droplet.

## Terminal

We need to set up terminal first, otherwise clear command and etc won't work over SSH frow Wayland session.

Add the following to `~/.bashrc` and source it:

```
export TERM=xterm-256color
```

## Setup SSH user

We don't want root login to be possible even via SSH.

- `adduser tyr` (use simple password, and skip the name)
- `usermod -aG sudo tyr` - add the user to sudo group

Edit `/etc/ssh/sshd_config`:

```
Port CUSTOM_PORT # if not set up yet at this point
PermitRootLogin no
PasswordAuthentication no
```

Update `/home/tyr/.ssh/authorized_keys` - add our publickey there, and remove it from the root folder:

```
sudo mv /root/.ssh/authorized_keys > /home/tyr/.ssh/authorized_keys
chown tyr:tyr /home/tyr/.ssh/authorized_keys
```

And then:

`systemctl reload sshd`

Update local ssh config and reconnect.

Update terminal again:

`echo "export TERM=xterm-256color" >> ~/.bashrc && source ~/.bashrc`

## Update the system

```
sudo apt update
sudo apt upgrade
```

## Install netdata

1. Sign in to NetData
2. Go to Nodes tab, click Add Node, copy the script and run it on the node

## Install wireguard & connect to mesh

- `sudo apt install wireguard`
- `mkdir security && cd security`
- `mkdir wg && cd wg`
- `wg genkey | tee privatekey | wg pubkey > publickey`

> I'm using the same listen port for WG mesh on all the nodes, just look it up on any one. It's not 50000.

Create `/etc/wireguard/wg0.conf`:

- We assume that we pick `10.8.0.5` IP address on the mesh for the new droplet (current machine)
- ListenPort & other peer ports should be exact ports that WireGuard is set up to listen to
- We assume that the droplet is created in the same datacenter as the main droplet (they'll be using private VPC addresses)
- Main droplet address is 1
- The addresses for worker nodes in main region start from 51 incrementally
- The addresses for second-region nodes start from 101
- My PC (ivanpc) address is 2

```
# do-worker-lon-1 <--- these kind of comments are helpful in this config, leave them
[Interface]
PrivateKey = <newdroplet_privatekey>
Address = 10.8.0.51/24
ListenPort = 50000

# do-main-lon
[Peer]
PublicKey = <maindroplet_publickey>
AllowedIPs = 10.8.0.1/32        # WG mesh IP address of maindroplet
Endpoint = 10.10.10.10:50000    # PRIVATE IP address of maindroplet (internal VPC)

oMxUz4aOkjopAS2Eu1nRaIwz8xbEWF6JP0U7tk04HXo= pri
6uDzh2ZGcRNwU73sAYYvQefAT8LPyVVBubgdhC8ISWk= pub

... same region droplets ^ similar config ...

# Different region droplet
[Peer]
PublicKey = <regiondroplet_publickey>
AllowedIPs = 10.8.0.101/32                  # WG mesh IP address of regiondroplet
Endpoint = regional.typingrealm.com:50000   # PUBLIC IP/domain address of regiondroplet (can't use VPC, different region)

... different regions droplets OR my own PC ^ similar config ...
```

This kind of config should be on every node/droplet/PC that should be in the mesh, synced together. So obviously this doesn't scale well, but we have small enough setup to support this.

**HOWEVER**, alternative solution is to have this kind of config on every "sub-node":

```
[Peer]
... MAIN DROPLET info ...
AllowedIPs = 10.8.0.0/24
```

When specifying `10.8.0.0/24` address - it means you can talk to ANY other droplets. Downside being - you are talking to them via the main droplet, so the main droplet becomes a single point of failure.

So, this being said, the configuration for any side-node can be as simple as:

```
[Interface]
...my info...

[Peer]
...main droplet info...
AllowedIPs = 10.8.0.0/24
```

And only the main droplet needs to have all the information about all the nodes in the mesh.

> I might migrate to the full mesh in future, given I have little amount of Droplets.

> `/32` means single IP, `/24` means whole subnet (any IP).

Now, restart WG (after editing config) on the main droplet:

```
sudo systemctl restart wg-quick@wg0
```

And start/enable it on the current droplet:

```
sudo systemctl start/enable wg-quick@wg0
```

> If you changes are erased after you restart wg-quick - check for `SaveConfig` property, it should be missing or be false, otherwise you need to configure peers using wireguard tool commandline instead of editing it manually.

## Install docker

Follow the following page: [Docker Debian install guide](https://docs.docker.com/engine/install/debian/) to install **Docker Engine** from **APT** repository (not Docker Desktop).

Add the user to docker group:

- `sudo usermod -aG docker tyr`
- `newgrp docker` & reconnect

## Copy infrastructure files

Copy necessary infrastructure files, before joining this node to Swarm:

- `/root/dp.pfx` - dataprotection key/certificate
- `/data/*` - any configuration data (`secrets.env` etc) for services

These files need to be copied manually and maintained consistent between all Droplets. In future we might migrate to some kind of centralized configuration/keyvault storage solution.

Example of commands:

- `scp sshalias:/data/projectname/{secrets,secrets-compose}.env .`
- `scp sshalias:/root/dp.pfx .`

And then uploading (assuming you create the project folder first):

- `scp {secrets,secrets-compose}.env sshalias:/data/projectname/
- `scp dp.pfx sshalias:/root/
- `rm dp.pfx {secrets,secrets-compose}.env`

> Adjust the commands for the lack of root user: first copy the files somewhere where your ssh user can grab them (and give them correct permissions), then copy the files over to some allowed location, ssh to the droplet and manually move the files & set the permissions.

> For now anyone can read them, figure out the correct user and set up proper permissions.

## Join to Swarm

On the leader node:

- `docker swarm join-token worker` - make sure IP address is the WireGuard IP

Then use whatever is printed on the new droplet (follower node), just add `--advertise-addr [wireguard IP]`.

Update label to add it as a worker to my network:

`docker node update --label-add worker=true NODE_NAME`

> At this point, Swarm will already deploy worker services to this node when possible.
