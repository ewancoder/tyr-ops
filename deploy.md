# Deploy

We can deploy the services using regular Docker Compose, or using Docker Swarm. It is better to completely move to Docker Swarm as it provides more benefits, but I'm still maintaining my old Docker Compose builds/deployments for the sake of being able to use them if need be.

## Docker compose

Docker compose file allows us to specify a list of services (infra, services) that need to be deployed together as a stack.

Advantages:

- You don't need a Docker Swarm setup.
- You can deploy the stack easily to any machine.

Disadvantages:

- Caddy requires static configuration for load balancing between the replicas: if you specify 3 replicas - you need to add 3 entries for the load balancing, specifically stating `service-1, service-2, service-3`.
- If you join your services to an overlay network (for single central Caddy to provide domain names for it, or for administrative services) - sometimes it doesn't work because overlay network might not be synced to your Swarm worker node unless you deploy the service in the Swarm mode (this is why I created Doneman as a hacky temporary solution to this problem).

## Docker Swarm

Docker Swarm allows you to deploy the service to any machine based on the configuration. Here's the flow:

1. You add more machines (nodes) to the Docker Swarm network
2. You add Docker Swarm labels to specific nodes
3. You specify in the configuration to which types of nodes particular services can be deployed
4. You deploy to the main (leader) node - Docker Swarm will take care of the rest!

Docker Swarm orchestrates deployment, i.e. it will deploy the services to the appropriate machines, granted these machines have necessary resources to host them, and it will re-balance them as machines become unavailable or available.

> Note that if some worker machine becomes unavailable, Docker Swarm will rebalance / deploy the missing services to other machines, but after this machine becomes available again - it will **NOT** automatically balance the services back to it.

Advantages:

- Caddy can automatically receive replica IPs (dnsrr) and load balance your requests between all the services, no need for static configuration
- No issues with deployments - networks are always synced appropriately between the nodes, because the Leader node is doing the deployment
- No need to expose SSH/SSHD on worker node / PC, for local deployment, everything is done by the Leader node over Docker Swarm network
