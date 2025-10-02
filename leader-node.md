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
