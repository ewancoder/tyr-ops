# DigitalOcean

This is a hosting service where I'm renting virtual machines (called Droplets).

## Creating a new Droplet

1. Select a region
  - London (LON1) - main region
  - New York (NYC2) - side region
2. OS - Debian, latest version
3. Type - basic (shared CPU, cheapest for playground purposes)
4. Best options at the time of writing:
  - $6 - cheapest, 25Gb storage, 1Gb ram
  - $8 - 35Gb nvme, 1Gb ram (currently used workers/other regions)
  - $12 - 50Gb storage, 2Gb ram
  - $14 - 50Gb nvme, 2Gb ram (currently used for prod)
    - Consider using the same for second region main node when going multi-region
  - $16 - 70Gb nvme, 2Gb ram

Additional features:

- Additional volume - 10$/100Gb
- Weekly backups - 1.6$
- Daily backups - 2.4$

> We don't need backups on worker/side nodes, as all the data is stored on the main node only.

5. Authentication options - SSH key (create for the new droplet)
  - Or later - add the public key to `~/.ssh/authorized_keys`
6. Add improved metrics monitoring and alerting - it's free, so let's test this
7. Do **not** enable ipv6 for now, let's do it manually later if needed.
8. Also no need for initialization scripts.
9. Hostname - `do-worker-region-ID`, for the first worker for example `do-worker-lon-1` etc.
  - Probably a good idea to also change hostname of main node, to `do-main-lon`.
10. No need for tags (for now).
11. Project - `TypingRealm`.

## Setting up ssh access

1. If not done during droplet creation - create SSH key and add public key to `~/.ssh/authorized_keys` on the droplet.
2. Setup local PC `~/.ssh/config` for connecting to the droplet: `Host do-main-lon, do-worker-lon-1, do-worker-nyc-2`.
3. SSH to the server initially using `22` port, edit `/etc/ssh/sshd_config` to change the Port, `systemctl restart sshd`, test that new port works

