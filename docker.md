# Docker

I'm using Docker for all my pet projects deployments, so this article will mostly contain my specific TyR setup.

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

For development environment, we store the data on the PC at: `/mnt/data/pet`, this folder is symlinked to `/data/pet` and deployments are done to `/data/pet/project`, not `/data/project`, when deploying to dev env, for convenience of management on personal PC.

> When copying configs between servers while moving between environments - make sure to actually edit `secrets.env` files for the specific environment, not to allow for example `dev` environment to write into `prod` database.
