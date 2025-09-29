# TyR cluster data

For TyR application, both docker files and services depend on `secrets.env` or `secrets-compose.env` files in the respective `/data/env-project` folders.

For regular docker compose deployments, we SSH directly to the machine where it should be deployed and deploy using `secrets.env` file. For Swarm deployments, we use `secrets-compose.env` file.

The differences between the two:

- Seq uri: `http://seq:5341` in compose, `http://tyr-infra-seq:5341` in direct. Not sure why, need to investigate / recall.
- `DbConnectionString/DATABASE_URL`: `appname_postgres` host in Swarm, `appname-postgres` host in regular Docker. Because we specify `-` manually there, but Swarm automatically uses `_`.
- `CacheConnectionString`: same, `-` delimiter for regular docker, `_` for Swarm.

> Currently I'm planning to switch over from having environment files to having Docker Swarm secrets, for the Swarm configuration.

## Secrets hints

- Password for DP (dataprotection) PFX certificate is a simple one, but with reg numbers.

## Docker Swarm secrets

Swarm secrets are better than env files because they are accessible cluster-wide and we do not need to copy them to every node.

We can create a separate secret for every value to have much more granular control, but this comes with tradeoffs of maintainability.

My current approach - saving the current `secrets-compose.env` file as a single secret so that it is shared across the cluster. APIs (or different backends) use it as a secret and read on startup to fill environment variables properly.

We still need to create at least one more separate secret for postgres database creation though.

### Secret names

Secrets are encrypted and only containers that mount them can read them. Do not store them in a Config because then anyone can inspect them (via docker inspect for example). We could afford storing them in files previously because we could restrict file permissions, but we can't restrict Docker Config.

- `tyr_{env}_{app}` - contains environment file with all necessary secrets for APIs to run, shared between APIs if we have multiple APIs
- `tyr_{env}_postgres_password` - postgres needs this in order to deploy the first time, so we have it as a separate secret to allow it to do just that
- `tyr_{env}_dp` - dataprotection pfx certificate for given environment

> See [dataprotection.md](dataprotection.md) on more info for TyR dataprotection setup.

> When creating a new service - copy existing `secrets-compose.env` file to a new folder and create a Swarm secret form it. When any secrets change - update the Swarm secret.

> We need to implement the code for reading these secrets directly in our .NET apps, because we use chiseled/distroless images and we do not have access to any kind of shell.

### Updating the secret

Secrets in Docker Swarm are immutable, so you cannot update it.

You also cannot remove it while any stack is using it.

So updating the secret is a major PITA in a Swarm cluster.

- `docker secret ls` - identify the secret you need to update
- `docker secret rm secretname` - see that it's being used
- `docker stack rm stackname` - delete stacks that use this secret
- `docker secret rm secretname && docker secret create secretname contentfile`
- Redeploy necessary stacks / pipelines

## Docker swarm configs

Swarm configs are non-sensitive files that are accessible cluster-wide, similarly to the secrets.

For example, we can have a script that reads environment variables from a secret file, one config value across all services, and we can use it as an entry point.

Unfortunately though, I'm using chiseled/distroless images for my .NET processes, so we do not have any shell available to us.
