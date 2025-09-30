# TyR cluster data

Since we are using the same Docker Swarm cluster for multiple environments, we need a clear name for every service, network, secret, or other resource.

- Scope - `SCOPE=tyr` - this is the scope of all our typingrealm apps
- All applications data folder: `DATA_FOLDER=/data` - we store all applications here, across many scopes if needed
  - Application data may include secrets, env variables, database volumes, redis volumes, etc.
- Deployment folder: `DEPLOY_FOLDER=/tmp` - we copy all deployments files here during the deployment, and delete after
- Application deployment folder: `APP_DEPLOY_FOLDER` = `$DEPLOY_FOLDER/$SCOPE/$ENV/$APP`
  - Example: `/tmp/tyr/prod/aircaptain`
- Application data folder: `APP_DATA_FOLDER` = `$DATA_FOLDER/$SCOPE/$ENV/$APP`
  - Example: `/data/tyr/prod/aircaptain`
  - This folder contains any data related to the application stack: postgres, valkey, etc
    - Example: `/data/tyr/prod/aircaptain/postgres` - postgres database volume
- Secrets folders:
  - Global: `$DATA_FOLDER/$SCOPE/$ENV/secrets`
  - App-specific: `$DATA_FOLDER/$SCOPE/$ENV/$APP/secrets`
- Service name: `$SCOPE-$ENV-$APP_$Service`
  - Example: `tyr-prod-aircaptain_api`, `tyr-prod-aircaptain_postgres`
  - Unfortunately we need to live with `_$Service` instead of a dash - hardcoded Swarm behavior
  - Unfortunately we cannot prefix the service resource with `-svc-` because the only way to do this is to rename the services themselves in the compose file (e.g. from `api` to `svc-api`) and I don't want to do this
- Network name
  - All networks should preferably be environment-level (isolated environments) so we can make sure `dev` cannot talk to `prod`
  - `$SCOPE-net-$NetworkName`: `tyr-net-proxy` (undesirable)
  - `$SCOPE-$ENV-net-$NetworkName`: `tyr-net-infra`
  - `$SCOPE-$ENV-net-$NetworkName`: tyr-prod-net-infra`
  - `$SCOPE-$ENV-$APP-net-$NetworkName`: tyr-prod-aircaptain-net-local`
- Secret name
  - `$SCOPE-$ENV-sec-$SecretName`: `tyr-prod-sec-dp`
  - `$SCOPE-$ENV-$APP-sec-$SecretName`: `tyr-prod-aircaptain-sec-secrets`

> By default, networks will be internal. So instead of postfixing them all with `internal`, we'll postfix the public ones with `public`.

> $APP comes always before `sec/net` (resource type), because resources are "nested" in the app hierarchy.

Special networks:

- `tyr-administration-internal` - for connecting services to administration apps (pgadmin, redisinsight). Will be migrated in future into:
  - `tyr-prod-net-administration`
  - `tyr-dev-net-administration`
- `tyr-infrastructure-internal` - for connecting services to logging systems & etc. In future:
  - `tyr-prod-net-infrastructure`
  - `tyr-prod-net-infrastructure`
- `tyr-proxy-internal` - used to connect services to reverse proxy. In future:
  - `tyr-prod-net-proxy`
  - `tyr-dev-net-proxy`
- `typingrealm` - old network for connecting non-swarm containers to the proxy, will be removed in future

Infrastructure (e.g. shared cache) naming convention:

- `$SCOPE-$ENV-infra-$APP` - e.g. `tyr-prod-infra_cache` for shared cache

> Currently we have a single deployment compose file for all environments, so the service name becomes `tyr-infra_prod-cache`. We achieve desired `tyr-prod-infra_cache` by using **Aliases** feature, including this alias to the infrastructure (and administration) networks.

> Seq name is just "seq" (achieved via aliases) because it's being used for ANY logs from ANY apps and ANY scopes. Possibly it's not a bad idea to have multiple Seq instances, but we are not going to implement it for the same reason we are not going to have a separate Swarm cluster for each environment: convenience and small scope.

> Seq also has multiple aliases, some of which are: `tyr-infra_seq`, `tyr-prod-infra_seq`, `tyr-dev-infra_seq`, so we can use a respective environment-specific seq alias and not worry if we decide to split the setup in future.

> Infrastructure environment-agnostic data is stored at `/data/tyr/infra`, environment-specific at `/data/tyr/$ENV/infra`.

> There are environment-specific networks: `tyr-net-proxy` for example, for the use by environment-agnostic services like pgadmin to be connected to proxy.

### Seq logging

Seq logging sets up the following data fields in its API Key application properties:

- `Application=CamelCaseBusinessAppName` (AirCaptain, FoulBot)
- `App = the same ^`
- `Service=tyr-prod-aircaptain_api` (full docker stack name)
- `Environment=Production/Development/etc`
- `Env = the same ^`

The name of the API key itself is a docker service name because it uniquely identifies separate API keys.

## Secrets

Every app needs access to its secrets: connection strings to databases, etc.

Previously we were storing secrets in the `secrets.env`/`secrets-compose.env` files in the application data folder `$APP_DATA_FOLDER`. This was working fine for regular docker compose deployments, and it was fine-ish for Swarm deployments because we have a little cluster. But it comes with a major downside: these files / configs need to be copied to **every** node in the cluster. When we add a new node, we always need to copy all the configuration files for all the services.

Docker Swarm provides a better mechanism for it: Docker Swarm Secrets. Sure, we can go with an external solution like some kind of KeyVault, but I wanted firts to learn what Docker Swarm can give us to manage this.

Secrets are also better because environment variables can be easily read by using `docker inspect`, whereas secrets can only be read by a container that uses them.

Read more about Docker Swarm Secrets in [docker.md](docker.md).

the content of previous `secrets.env`/`secrets-compose.env` file in the secret:

- `$SCOPE-$ENV-sec-$APP-secrets` - service specific secrets
- `$SCOPE-$ENV-sec-secrets` - global secrets (shared across environment)

Secrets are stored in their respective folder in plain text, for versioning:

- `/data/tyr/prod/secrets/secrets` - global secrets
- `/data/tyr/prod/appname/secrets/secrets` - service-specific secrets, etc.
- `/data/tyr/prod/appname/secrets/something-specific` - something-specific secret of appname app

> We still need to have `/data/tyr/prod/appname/secrets.env` file containing a single secret - `DATABASE_URL=xxx`, for `dbmate` migrations to work.

## Data structure

For TyR application, both docker files and services depend on `secrets.env` or `secrets-compose.env` files in the respective `/data/env-project` folders.

For regular docker compose deployments, we SSH directly to the machine where it should be deployed and deploy using `secrets.env` file. For Swarm deployments, we use `secrets-compose.env` file.

The differences between the two:

- Seq uri: `http://seq:5341` in compose, `http://tyr-infra-seq:5341` in direct. Not sure why, need to investigate / recall.
- `DbConnectionString/DATABASE_URL`: `appname_postgres` host in Swarm, `appname-postgres` host in regular Docker. Because we specify `-` manually there, but Swarm automatically uses `_`.
- `CacheConnectionString`: same, `-` delimiter for regular docker, `_` for Swarm.

> Currently I'm planning to switch over from having environment files to having Docker Swarm secrets, for the Swarm configuration.

> We still need `secrets.env` file for secrets on `domain`: one in `app_dev`, one in `app_prod` folder, unfortunately, for dbmate deployment. Later when we move to packaging the migrations & running them as part of Swarm deployment - we can get rid of it completely.

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

Unfortunately this is a major PITA and leads to a **DOWNTIME**. So the only other solution is:

1. Create a new secret with name `secret-1`, `secret-2`, etc (versioning)
2. Update yaml file to use the updated secret (maybe using .env variables)
3. Redeploy the stack with the new secret

#### Current secret management

We are iterating over all the secrets that are defined in the compose file as part of deployment process. We check the secret in Docker Secrets for every secret - and get the latest version (numerically ordered). And we substitute this secret version into the compose file before deployment.

This way we make sure we always use the latest secret version.

## Docker swarm configs

Swarm configs are non-sensitive files that are accessible cluster-wide, similarly to the secrets.

For example, we can have a script that reads environment variables from a secret file, one config value across all services, and we can use it as an entry point.

Unfortunately though, I'm using chiseled/distroless images for my .NET processes, so we do not have any shell available to us.

## New (leader) cluster setup

With time, it will contain complete list of commands to recreate the cluster. For now it's a limited set.

```
echo tyr-prod-net-administration tyr-prod-net-infrastructure tyr-prod-net-proxy tyr-dev-net-administration tyr-dev-net-infrastructure tyr-dev-net-proxy | xargs -n1 docker network create --driver overlay --attachable --internal
```

## New service creation

### Service networks

> We can skip specifying `driver: overlay` because Swarm (stack) services always have all networks as overlay.

- Defined local network (alias `local`): `$SCOPE-$ENV-net-$APP` - `tyr-prod-net-aircaptain`
  - This network is used for backend services to communicate with eath other and with the infrastructure. For example, .NET API, Go Backend, Postgres, and Redis, all should be on the same network
- Defined local network (alias `deploy`): `$SCOPE-$ENV-net-$APP-deploy`
  - used for deployment operations - like applying DB migrations: so `postgres` service can be on this network, and we connect to it directly from the deployment script (or another ephemeral service)
  - It should be attachable, so that we can connect to it during deployment (unless ephemeral service is used)
- `proxy`, `infra`, `admin` networks - our special cluster-wide (environment-wide) networks
- `internet` network - without name & config - name generated by Swarm
  - used just to provide ability to open ports to host system on development env (for database/cache/etc access for development)

### Service secrets

- `tyr-${ENV}-sec-postgres-password` - default postgres password
- `tyr-${ENV}-sec-aircaptain-secrets` - aircaptain application secrets env file
- `tyr-${ENV}-sec-dp` - data protection certificate

#### API service secrets.env content

Previously all the secrets were stored together in the `secrets.env` file. Now we store them in docker secrets.

These are the secrets for the TyR infrastructure, that should be separate for each service:

Stored in: `tyr-$ENV-sec-$APP-secrets` secret.

- `SeqApiKey` - API key for this specific App for Seq ingestion
- `DbConnectionString` - Connection string to the database
- `DATABASE_URL` - Connection string to the database, in the format for Database Migrations (dbmate)
  - This secret can be moved away after we migrate to ephemeral services, and it's only needed in the env file, not in the Docker secret
- `POSTGRES_PASSWORD` - This secret was used before for specifying the default Postgres password
  - Now we are specifying it using a separate cluster-wide secret, so we can remove it from the secrets.env
- `CacheConnectionString` - Service-specific cache connection string for anything that needs distributed cache

These are the secrets that are still there, but can be moved out to "global" env secrets:

Stored in: `tyr-$ENV-sec-secrets` secret.

- `SeqUri` - URL for Seq logs ingestion
- `DpCertPassword` - Password for data protection certificate
- `GlobalCacheConnectionString` - Connection string to the Global cache (shared between services of one env, for things like data protection)
