# Geo-based load balancing

Load balancing provides a way for us to have multiple regions with the following ideas:

- If user lives in USA - we route traffic to USA region, if user lives in Europe - we route traffic to Europe region
- If user lives in USA, but USA region services are unhealthy - we route the traffic to Europe to avoid downtime

For geo-based load balancing we can go with two options described below.

## Digital Ocean load balancer

Advantages:

- Integrates well with existing Droplets infrastructure

Disadvantages:

- Works only with internal (digital ocean) resources

## CloudFlare load balancer

Advantages:

- Abstracted away, can work with ANY IPs (be it a DO, or other infra)
- A lot more locations (250+) coverage / 13 geo regions

For CloudFlare LB, we need to enable **Traffic Steering**, as this is required for routing traffic between geo regions, otherwise our LB will only work for failover.
