# Data Protection

Previously with Docker Compose, everything was hosted on the same machine, so we had folders/filesystem for dataprotection.

Now that we are using Swarm, we need to migrate this approach.

## Data Protection certificate

We can store data protection certificate in Docker Swarm Secret: `tyr_{env}_dataprotection`.

## Data Protection keyring storage

Keys from one pod should be accessible by another pod, in order for other pods to be able to validate the Cookie / anything else that we might have data-protected.

Furthermore, since we are considering all our TyR apps using the same `typingrealm.com`/`typingrealm.org` domain, and I want them all to share cross-domain working cookies, we need to ensure all our TyR apps have the same keyring for data protection.

This is why we need to spin up a separate, cluster-wide, `redis`/`valkey` container, that will be used for just that.

As an option, we might go with some kind of keyvault solution, or store them in mounted network filesystem in Swarm cluster.

For complete `TyR` cluster-level infrastructure project documentation, refer to [tyr-infra.md](tyr-infra.md) or [tyr-infra repository](https://github.com/ewancoder-tyr/tyr-infra).
