# Keys

## My keys inventory

- Git (gpg) - main key for signing commits for personal projects, including signing subkey
  - Master ID: 98B0943EC4A3CA88
  - Signing ID (ivanpc): A57116EE67BF8798
  - Public key is uploaded to GitHub
- NameOfWorkCompany (gpg) - signing commits for work, including signing subkey
  - Master ID: 689985CE63F245C9
  - Signing ID (ivanpc): D6335448AD610A55
  - Signing ID (ivanlaptop-win): 2359493617E1C282
- Main PC SSH keys (multiple aliases can be specified like this: `Host do-main-lon domain`
  - domain - (do-main-lon) connection to main DigitalOcean droplet (comment: ewancoder@ivanpc-domain)
  - doworker - (do-worker-lon-1) connection to worker DigitalOcean droplet
  - github - personal github account
- TyR infra SSH keys (stored on Main PC)
  - github2domain - connection from github actions to do-main server
    - comment: ewancoder@github-domain
    - file: `..._github2domain`
    - storage: `~/.ssh/remote` folder (not in root)
    - Used for all repos deployment. I can't be arsed to have a separate ssh key for each project.
  - github2dev - connection from github actions to dev PC (ivanpc)
    - It is added to `authorized_hosts` of `github` user on my machine - specific user for deployments
- Laptop SSH keys
  - (workname) - connection to work repositories
    - RSA `-t RSA` unfortunately - Azure doesn't support ED
    - Comment: "workusername@ivanlaptop-workname"
- Work VDI
  - SSH doesn't work there unfortunately, using HTTPS authentication
- PATs - personal access tokens / access tokens for different systems
  - Work - (security)/work
    - deployment-monitor-pat
      - 1 year expiration (21/09/2026)
      - wiki read & write
      - release read
      - build read
- TyR other keys
  - DP - data protection PFX key
    - Currently stored at `/root/dp.pfx`, should be copied to every droplet / be the same
    - Currently copies are only on droplets, not on my machine

## Security

Edit `/etc/ssh/sshd_config` `Port` property to specify custom port, both on servers and on my PC.

All keys (both GPG and SSH) should have 700/600 permissions so that nobody else can view them but you.

Easy way to ensure this:

- `chmod -R u=rwX,go= ~/.ssh`
- `chmod -R u=rwX,go= ~/.gnupg`

Uppercase `X` ensures that executable permission is given only to:

1. Folders
2. Files that already had executable permission anyway

`go=` ensures that `groups` and `others` permissions are set to zero.

## Algorithms

- RSA - old, slow, big size
- ECC (Ed25519) - new, fast, small size

The only downside of ECC is that it might not be supported everywhere.

I'm sticking with ECC for my keys.

## GPG

GPG - GNU Privacy Guard - a tool for encrypting, decrypting, and signing data.

- Altertative of proprietary software PGP (pretty good privacy), started in 1997.
- Made by Werner Koch (German).

Usage scenarios:

1. You share public key with someone, they can encrypt a message to you using it, and you can read it by decrypting it by using private key.
2. You sign some data with your private key, and people can verify it with your public key.

My locations:

- (security)/gnupg - symlinked to home/.gnupg
- (security)/backup - backup of master keys (exported for future import)

Use difficult unique passwords for specific keys (use comment/purpose of the key as salt).

Consider using different subkeys for different machines - this allows checking later which machine actually did commits (assuming I have a mapping between key ID -> machine).

### Generating

- `gpg --full-generate-key`
- `ECC (sign and encrypt)`
- `Curve 25519`
- `0` - never expire
- Comment: `Git` (for git master key)

The key will be stored in `~/.gnupg`.

Check the key:

`gpg --list-secret-keys --keyid-format=long`

The part after `/` is the key ID. SEC - master key. SSB - subkeys.

### Adding a subkey

- `gpg --edit-key MASTER_KEY_ID`
- `addkey`
- `ECC sign only, 25519, 0` (for signing git commits)
- `save`

### Backing up all keys (master export)

- `gpg --export-secret-keys --armor MASTER_KEY_ID > git.asc` (`git.asc` is the name of the file)

### Exporting public key

This is needed for uploading the public master key to e.g. GitHub to get a verified status.

`gpg --armor --export MASTER_KEY_ID`

### Exporting a signing subkey to another machine

- `gpg --export-secret-subkeys SUBKEY_ID! > subkey.gpg`
- then on another machine: `gpg --import subkey.gpg`

If you need to also be able to verify signatures on second machine, you need a public key as well:

- `gpg --export --armor MASTER_KEY_ID > publickey.asc`
- `gpg --import publickey.asc`

In order to not see "unknown/untrusted" messages in git log, do the following:

- `gpg --edit-key MASTER_ID` (will work even without exporting a public key)
- `trust`, select 5 (ultimate trust) for own key
- `quit`

## SSH

For securely connecting to a shell, or forwarding ports (tunneling).

My locations:

- (security)/ssh - symlinked to home/.ssh

Use simple password for SSH keys (just so that there IS password).

### Generating

- Default algorithm is ED25519.

Comment can be tweaked later, but it requires changing both public and private keys. Default comment is usually "username@hostname".

The default comment identifies WHO and FROM WHERE you are trying to connect. But it doesn't identify TO WHERE you are trying to connect.

This is fine in general, because usually you reuse the same SSH key for multiple different systems. However, for more granular control (if you make different keys to access different systems), it makes sense tweaking the default comment a bit:

- `username@hostname-target`
- `ivan@laptop-domain`
- `ivan@laptop-github`

Filename pattern: `id_ed25519_postfix`, e.g. on ivan@laptop machine, to connect to github: `id_ed25519_github`.

Since we do not include the "who" in the name of the key - keys should not be shared between machines (each machine generates its own).

If we ever need to centrally store these keys - these files should be renamed to specify the "who".

To generate a keypair:

- `ssh-keygen -C "comment"`
- `ssh-keygen -C "user@host-purpose" (ewancoder, ivanpc, domain/github)
  - filename: id_ed25519_domain/github

### Connecting

#### Adding entry to local known hosts

`known_hosts` file protects from man-in-the-middle attacks. During initial connect, the host sends you "host fingerprint", and if it doesn't exist in your local `known_hosts` file - SSH asks you whether the fingerprint is correct.

To check the fingerprint on the host server:

`ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub`

#### Adding entry to remote authorized keys

`authorized_keys` on the remote host should have your public keypair inserted into it, in order for you to be able to connect to it using your private key.

Copy your public (pub) key, and append it as a line into `~/.ssh/authorized_keys` file on the remote server.

#### Multiple keys

If you have multiple SSH keys, you need to create a configuration file `~/.ssh/config` to specify which keys to use for which servers.

Create this `~/.ssh/config` file:

```
Host somename # somename is your custom alias for this entry
    HostName myserver.com # or IP address
    Port 22 # or custom port
    User root # or rootless user
    IdentityFile ~/.ssh/id_my_custom_file # path to the PRIVATE key
```

In order to connect to this entry, you need to specify your custom alias when connecting:

`ssh somename`
