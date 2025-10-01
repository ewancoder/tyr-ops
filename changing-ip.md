# Changing IP guide

If we change an IP address of a domain `typingrealm.ORG`, but Caddy is set up for Let's Encrypt for the domain `typingrealm.COM` (and the IP addresses for these two domains, even though pointing to the same physical machine, are still different) - then SSL cert validation will fail for the new IP / `typingrealm.ORG`.

The solution is to also update `typingrealm.COM` to the new IP address.
