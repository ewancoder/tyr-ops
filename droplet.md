# Droplet setup

This is a complete sequence of all the things I'm doing to set up a freshly added worker droplet.

## Terminal

We need to set up terminal first, otherwise clear command and etc won't work over SSH frow Wayland session.

Add the following to `~/.bashrc` and source it:

```
export TERM=xterm-256color
alias sudo='run0'
```

> Add it at the very beginning of the ~/.bashrc, otherwise console won't be colored after initial login.

## Setup SSH user

We don't want root login to be possible even via SSH.

- `adduser tyr --uid 2000` (use simple password, and skip the name, don't forget to specify 2000 id)
- `usermod -aG sudo tyr` - add the user to sudo group

Update `/home/tyr/.ssh/authorized_keys` - add our publickey there, and remove it from the root folder:

> If setting up a **LEADER** node - also add `github2domain` public key to `authorized_keys`, so that github can SSH to it.

```
sudo mv /root/.ssh/authorized_keys > /home/tyr/.ssh/authorized_keys
chown tyr:tyr /home/tyr/.ssh/authorized_keys
```

Edit `/etc/ssh/sshd_config`:

```
Port CUSTOM_PORT # if not set up yet at this point
PermitRootLogin no
PasswordAuthentication no
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

- Reboot after the upgrade (in case kernel is updated)

## Install netdata

1. Sign in to NetData
2. Go to Nodes tab, click Add Node, copy the script and run it on the node

## Install wireguard & connect to mesh

The addresses for machines that can be used instead of IPs:

- `domain1.swarm.typingrealm.org` - leader node
  - Alt: `ssh.typingrealm.org, typingrealm.org`
- `worker51.swarm.typingrealm.org` - first worker node
  - BUT: Use in-cluster IP address instead, to route the traffic via datacenter
  - Alt: `worker.ssh.typingrealm.org`
- `ivanpc2.swarm.typingrealm.org` - my pc / dev machine
  - Alt: `ivanpc.typingrealm.org`

> The numbers are the IP addresses that we have in the network.

Run these:

- `sudo apt install wireguard`
- `mkdir security && cd security`
- `mkdir wg && cd wg`
- `wg genkey | tee privatekey | wg pubkey > publickey`

> I'm using the same listen port for WG mesh on all the nodes, just look it up on any one. It's not 50000.

Create `/etc/wireguard/wg0.conf`:

> If setting up a new LEADER node - copy the config from the previous leader node, containing all the records of all the nodes. Also do not forget to update ALL configurations of all other worker nodes to point to the new leader.

> `ivanpc.typingrealm.org` - my machine PC, can be used instead of the IP

### Allow IP forwarding

This step needs to be done on a NEW LEADER node, because by default it will not forward traffic between nodes: My PC won't be able to talk to Worker1 for example.

- `sudo /usr/sbin/iptables -A FORWARD -i wg0 -o wg0 -j ACCEPT`
- `sudo iptables -A FORWARD -o wg0 -i wg0 -j ACCEPT`
- `sudo apt install iptables-persistent` - and agree on saving current iptables rules
  - Alternatively: `sudo iptables-save > /etc/iptables/rules.v4`
- Reboot to check that it's persisted (pings are working between Worker nodes / PC)

### Example of Leader machine config

```
[Interface]
PrivateKey = Leader-PrivateKey
Address = 10.8.0.1/24 # Leader machine WG IP address.
ListenPort = 50000

[Peer]
PublicKey = Machine1-PublicKey
AllowedIPs = 10.8.0.2/32 # Specific WG IP address of this machine.
Endpoint = Machine1-IP # Public internet-accessible IP address of this machine, OR in-datacenter IP address when both Leader and Machine1 are in the same datacenter.

# ... machine2 machine3 ...
```

### Example of Worker machine config

```
[Interface]
PrivateKey = ThisWorker-PrivateKey
Address = 10.8.0.51/24 # This worker WG IP address.
ListenPort = 50000

[Peer]
PublicKey = Leader-PublicKey
AllowedIPs = 10.8.0.0/24 # This makes sure ALL traffic is routed through the Leader machine. Easier to maintain than a full mesh.
Endpoint = Leader-IP # Public internet-accessible IP address of this machine, OR in-datacenter IP address when both Leader and ThisWorker machines are in the same datacenter.
```

Now, restart WG (after editing config) on the main droplet:

- `sudo systemctl restart wg-quick@wg0`

And start/enable it on the current droplet:

- `sudo systemctl start/enable wg-quick@wg0`

### Additional info

> This info was written previously as a knowledge base, and unless you need a knowledge refresh - this section can be skipped, assuming you created configs based on the examples above.

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

> If you changes are erased after you restart wg-quick - check for `SaveConfig` property, it should be missing or be false, otherwise you need to configure peers using wireguard tool commandline instead of editing it manually.

## Install docker

Follow the following page: [Docker Debian install guide](https://docs.docker.com/engine/install/debian/) to install **Docker Engine** from **APT** repository (not Docker Desktop).

Add the user to docker group:

- `sudo usermod -aG docker tyr`
  - with `run0` unfortunately: `sudo /usr/sbin/usermod`, or use `sudo`
- `newgrp docker` & reconnect

## Copy data

If spinning up a new (upgraded) leader node - copy all the necessary data over (`/data` folder), otherwise worker nodes do not rely on any data (we store everything in Docker Swarm Secrets, so everything is accessible throughout the cluster).

## Install utilities

- `yq` - needs to be installed for my scripts automatically using latest version of the Docker Secrets.

> We need it only on the Leader node. And maybe in future we can just refactor use `jq`.

## Swarm configuration

### Create a new Swarm

> If setting up a new (instead of the old one) Leader node, then do this, otherwise join to the existing Swarm from the Worker node.

> If setting up a new (instead of the old one) Leader node, then go to all worker nodes and join them to this leader node.

- `docker swarm init --advertise-addr WG_LEADER_IP`

### Join to Swarm

On the leader node:

- `docker swarm join-token worker` - make sure IP address is the WireGuard IP

Then use whatever is printed on the new droplet (follower node), just add `--advertise-addr [wireguard IP]`.

Update label to add it as a worker to my network:

`docker node update --label-add prod-worker=true NODE_NAME`

> At this point, Swarm will already deploy worker services to this node when possible.

### Add necessary labels

For development env:

- `tyr-dev-infra=true`
- `tyr-dev-worker=true`

For production env:

- `tyr-prod-infra=true`
- `tyr-prod-worker=true`

Workers should only have the worker label. Infra should only be a single node (we store data on it).

### Re-join new swarm

To re-join the new swarm:

- `docker swarm leave`
- Edit /etc/wireguard/wg0.conf for the new network, restart wg0.
- Join the swarm as a worker node
- Set up necessary labels

## Reboot

Reboot after whole setup, just to make sure everything is properly restarted.

## If setting up a leader node

Check out the rest at [leader-node.md](leader-node.md).
