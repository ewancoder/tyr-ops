# Caddy

Caddy is a reverse proxy with automatic (Let's Encrypt) certificate rotation.

This guide show my configuration for TyR projects (Caddyfile).

First section should include an email for Let's Encrypt to acquire certificate for you:

```
{
    email typingrealm@gmail.com
}
```

## Load balancing

This is a simple load balancing template for the services running in a regular docker mode:

```
(lb) {
    reverse_proxy {args[2:]} {
        lb_policy least_conn
        health_uri {args[0]}
        health_port {args[1]}
        health_interval 15s
        fail_duration 30s
        lb_retries 3
    }
}
```

- We take all arguments starting from the 3rd one as hosts that it will monitor.
- We set up `least_conn` load balancing policy - requests will be sent to the pod with the least connections.
- `health_uri`, `health_port` - healthcheck of a pod, this URI should return 200.
- `health_interval` - how often do we do healthchecks.
- `fail_duration` - how long do we wait until trying again, after the pod is considered unhealthy.
- `lb_retries` - how many times we retry different pods: if this is zero, we instantly return an error, otherwise we try doing the same request on another pod.

Assuming we have two containers `my-container-1` and `my-container-2` of our service, running on port 8080 with healthcheck accessible on /health url, we could use it like this:

```
www.something.domain.com {
    import lb health 8080 my-container-1 my-container-2
}
```

Where `my-container-1` and `my-container-2` are the container names of replicas of some service. Then, any request coming to `www.something.domain.com` will come to either the first one, or the second one, based on the load balancing policy. `round_robin` is another popular policy that just forwards the requests to all replicas equally in turns, just iterating over them.

### Docker Swarm DNS resolution

Docker Swarm has its own DNS resolution: for example, when you run a service with 3 replicas, you get service.1, service.2, service.3 containers, and when you ping a 'service' domain name - you get round-robined between 1st, 2nd, or 3rd replica.

However, Docker Swarm does not support more complicated load balancing, setting up healthchecks / different policies, etc. This is why we need to **disable** Docker Swarm load balancing and use our own, Caddy-level load balancing.

In order to do that, we switch the service's DNS into `dnsrr` Docker Swarm mode so that Docker does not do its own load balancing, but instead provides us with metadata about all pods of a specific service, and we can use that metadata on the Caddy level to have our own load balancing there.

This is an example of my load balancing configuration for swarm services, leveraging dnsrr metadata:

```
(lbswarm) {
    reverse_proxy {
        dynamic a {
            name tasks.{args[0]}
            port {args[1]}
            refresh 30s
        }

        lb_policy least_conn
        health_uri {args[2]}
        health_port {args[1]}
        health_interval 15s
        fail_duration 30s
        lb_retries 3
    }
}
```

> Docker swarm gives you a list of IP addresses of all the pods, if you query the hostname in this pattern: `tasks.servicename`, so if you service container name is `myapp`, you would query `tasks.myapp` domain name to get the total list of IP addresses of all the pods.

- `dynamic a` - tells Caddy to use dynamic upstream discovery feature, which will query DNS A/AAAA records of a given service. We are using `tasks.{args[0]}` here, so we need to pass just our service name as the first parameter.
  - `port {args[1]}` - as a second parameter we are passing port of our pods: this is also needed for caddy domain resolution.
  - `refresh 30s` - how often Caddy is going to re-resolve DNS name to get the list of upstream IPs.
- The rest of parameters are the same as in the example above - we setup our load balancing policy for the dynamically acquired upstreams.

Assuming we have two replicas `my-container.1` and `my-container.2` of our service named `my-container`, running on port 8080 with healthcheck accessible on /health url, we could use it like this:

```
www.something.domain.com {
    import lbswarm my-container 8080 health
}
```

## Further templating

To save us the trouble of entering health urls and ports every time, especially because all my services use the same stack and the same backing code / infrastructure, I created some helper templates as well:

```
(api) {
    import lb health 8080 {args[:]}
}

(web) {
    import lb / 8080 {args[:]}
}

(apiswarm) {
    import lbswarm {args[0]} 8080 health
}

(webswarm) {
    import lbswarm {args[0]} 8080 /
}
```

Both my API and Web pods are running on 8080 internal docker port, so we set up healthchecks to use that. For web applications, we just get the root path `/`, for APIs, we get the `/health` endpoint (this endpoint is implemented in my .NET hosts as a healthcheck endpoint).

We can use these templates like this:

```
www.something.domain.com {
    import apiswarm my-container
}

www.somethingelse.domain.com {
    import api my-container-1 my-container-2
}
```

However, during the actual configuration, I noticed that I need to have many similar URLs configured for each service, for example:

```
www.something.com,
something.com,
www.something.org,
something.org
```

Because I have both `com` and `org` domain, and because I want my services to be reachable from URLs both with `www` and without.

This is why I created ultimate templates for the swarm services (example is for prod-api, but there's also prod-web, dev-api and dev-web):

```
(prod_api_swarm) {
    api.{args[0]}.typingrealm.com,
    api.{args[0]}.typingrealm.org,
    www.api.{args[0]}.typingrealm.com,
    www.api.{args[0]}.typingrealm.org {
        import apiswarm {args[1]}
    }
}
```

And now we can use it like this:

```
import prod_api_swarm partofurl myservicename
import prod_api_swarm aircaptain prod-aircaptain_api
```

So that on the url `https://www.api.partofurl.typingrealm.com` you will reach `myservicename` service (but also on any similar urls, with/without `www` and with `.com` or `.org`).

## Security

Sometimes we want to protect endpoints that otherwise do not have authentication, like **RedisInsight** server that allows connecting to our Redis instances.

Caddy provides a way for basic authentication.

1. Create a password hash: `caddy hash-password --plaintext 'my-password'`
2. Insert the following section in your endpoint declaration:

```
... {
    basicauth {
        username password_hash
    }

    reverse_proxy ...
}
```
