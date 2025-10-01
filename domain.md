# Domains

Current domains are configured as raw A/AAAA records, without any CloudFlare features like DDOS protection etc. Because I didn't figure out yet how to make CloudFlare happy with Caddy Let's Encrypt-ion and other tcp/udp services.

- $MAIN_IP - my reserved floating IP on Digital Ocean that we assign to the main leader Swarm node where Caddy proxy is hosted
- $WORKER_IP - the IP of my first worker node (do-worker-lon-1)

Current domain names:

- *typingrealm.com - $MAIN_IP
- ssh.typingrealm.org - $MAIN_IP, not a separate record - I'm just using this domain for SSH (for future, if we decide to use CloudFlare proxy features for other domains)
- worker.ssh.typingrealm.com - $WORKER_IP