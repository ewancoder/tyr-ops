# Leader Swarm node

This is an additional guide about additional steps needed to set up the new (replacing) Leader node.

## Networks

Since we need our `Karting` application running on a non-Swarm cluster, we need to create a bunch of tyr-specific (legacy) networks.

- `typingrealm` - this network was used for regular apps. Create it as an overlay attachable network, so that we can run Caddy as a Swarm service. But not internal, since legacy apps might need internet.
    - `docker network create --driver overlay --attachable typingrealm`
