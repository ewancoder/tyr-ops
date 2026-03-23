# Reinstalling the Asgard server

## Base packages

1. Reinstall using **real**
2. Install/Update/Check that XPlane is working / link license
3. Link license / install XPME

## Media server

1. Clone media server (lab)
2. Start it, follow instructions (like adding Jellyfin API key)

## Pet projects development environment

### Join the node into Wireguard network

1. Add Asgard to the TYR wireguard network

- Generate private/public key pairs:

```
cd /mnt/data/security/wg
wg genkey > private
wg pubkey < private > public
```

- On **do-main**, `vim /etc/wireguard/wg0.conf`, add a peer (we can skip IP address)
  - Restart wg-quick@wg0

- On **asgard**, `vim /etc/wireguard/wg0.conf`, add main server same as on the wg-worker on DO, endpoint - `swarm.typingrealm.org`
  - `systemctl enable/start wg-quick@wg0`
  - `ping 10.8.0.1 to test that it's working`
  - `ping the node IP address from do-main to test the other way around`

### Join the node into Swarm cluster

- `docker node rm previous-node` - remove current dev node
- On **do-main**: `docker swarm join-token worker`
- On **asgard**: enter this
- Add necessary labels:
  - `docker node update --label-add tyr-dev-infra=true/tyr-dev-worker=true asgard`
