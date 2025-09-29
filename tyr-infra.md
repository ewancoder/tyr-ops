# TyR infrastructure

You can find [tyr-infra repository here](https://github.com/ewancoder-tyr/tyr-infra).

This document describes the thought process and decisions made when spinning up cluster-level tyr-infra infrastructure.

## Permissions

- We specify user: `2000` (this is our `tyr` user) for every container, because folders are created with this user.
- `pgadmin` is required to run as `5050` user, so we `chown` the directory for it.

## Environments

Every environment should be completely isolated from each other. Even though sometimes we might think that it would be beneficial having a shared Redis instance across all environments, or sharing a configuration variable - we cannot afford accidentally writing data from a development environment container to production database. This is why we take steps to ensure that every environment are using their own containers/configs/secrets, even if they have the same (duplicated) values.

## Caddy reverse proxy

- `tyr-infra-caddy`

We use Caddy reverse proxy for reverse proxying & load balancing (see [caddy.md](caddy.md)).

Caddy is a special service because it needs to both support Swarm services and non-swarm services, so we start it as a regular non-swarm service separately from the `tyr-infra` stack.

The important information:

- Caddy should be attached to `typingrealm` network. This is the main network to which all "past" services have been attached, otherwise they will lose connection to the Caddy. Some services that are still deployed in old single-node Compose mode are only attached to this network. This is a legacy network and will be removed after we move all the services to Swarm.
- `tyr-proxy-internal` - main TyR network, all the services should be attached to it from all environments, so that the Caddy can route the traffic as needed.
- It should only be deployed to the main DO node and mount necessary volumes for Caddyfile / data.

## Seq

- `tyr-infra-seq`
- Folder (root user: 0) - `/mnt/data/tyr-infra/tyr-infra-seq`

We use Seq to gather logs from all our services.

- It should expose necessary port for log ingestion.
- It should be deployed to `dev`/`IvanPC` machine because it requires a lot of resources.
- It should be connected to `tyr-infrastructure-internal` network. Any services that require logs to be sent to Seq will need to be connected to it as well.

## RedisInsight

- `tyr-infra-redisinsight`

This application allows viewing Redis servers easily, but it comes with a huge downside: no authentication.

So I'm still on terterhooks about using it.

- It should be connected to `tyr-administration-internal` - any services that need manual administration will be connected to it.

Redis Insight does not have any authentication features, so we need to protect it on the reverse-proxy level. Fortunately, Caddy provides such capabilities. See **Security** section of [caddy.md](caddy.md).

## PgAdmin

- `tyr-infra-pgadmin`

PostgreSQL admin client - web version. This is a great app including MFA capabilities.

- It should be connected to `tyr-administration-internal` - any services that need manual administration will be connected to it.

## Shared redis server

- `tyr-infra-prod-cache`
- `tyr-infra-dev-cache`

The folder with the data is respectively: `/data/tyr-infra/{prod,dev}-cache/data`. Development cache instance is running on the dev/pc machine.

Sometimes we need all our services to share data. Configuration and secrets can be safely distributed using Docker Swarm Config/Secret resources, but dynamic data cannot be easily shared.

For dynamic data (for example, data protection keyring) we deploy a shared `valkey` (redis) instance with persistence enabled.

> These caches - as any other cross-service resources (and not tools) should always be environment specific. Maybe it's even a better idea to split these caches to separate docker compose yamls in future. We just don't have many resources like this yet, just the cache, so for convenience we put it in the same Compose file and duplicate for 2 envs.
