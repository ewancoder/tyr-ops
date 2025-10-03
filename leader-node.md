# Leader Swarm node

This is an additional guide about additional steps needed to set up the new (replacing) Leader node.

## Networks

Since we need our `Karting` application running on a non-Swarm cluster, we need to create a bunch of tyr-specific (legacy) networks.

- `typingrealm` - this network was used for regular apps. Create it as an overlay attachable network, so that we can run Caddy as a Swarm service. But not internal, since legacy apps might need internet.
    - `docker network create --driver overlay --attachable typingrealm`

Create cluster env-scoped and global networks, denoted in tyr-cluster.md.

## Folders

- Prod: `/data/tyr/prod/infra/cache` under `tyr` user
- Dev: `/data/tyr/dev/infra/cache` under `dev` user
- Dev: `/data/tyr/infra/{pgadmin,redisinsight,seq}` - make sure they exist

- Copy caddy data: `/data/caddy` (including `/data/caddy/data`, `/data/caddy/Caddyfile`).

## Files

In `~/security/dp/dp.pfx` place dataprotection certificate.

## Secrets

Create all necessary secrets:

- `tyr-prod-sec-dp`
- `tyr-prod-sec-env`
- `tyr-prod-sec-postgres-password`

(same for dev)

For specific apps:

- `tyr-prod-APP-sec-env`

## Copy over helpful scripts

- `~/scripts` folder of the tyr user
  - `rebalance.sh` and `fast-rebalance.sh` - to rebalance the nodes after node reboot / add
  - `reload-caddy.sh` - for reloading Caddy config after changing it with zero downtime

Set up cron jobs:

1. SKIP FOR NOW. Rebalance: insert the following into `sudo crontab -e`: `0 0 * * * /home/tyr/scripts/rebalance.sh >> /tmp/rebalance.log 2>&1`
2. Backups: WIP (we want to backup /data folder and tyr ~/ home folder, to at least 2 other nodes)

Rebalance takes a long time, and is resource consuming. But it's good to run it in case we restart nodes. However, since we have just a few nodes, we can afford running the script manually if we reboot it. So just run `~/scripts/rebalance.sh` when needed.
