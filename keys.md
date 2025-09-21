# Keys

## Security

All keys (both GPG and SSH) should have 700/600 permissions so that nobody else can view them but you.
Easy way to ensure this:

`chmod -R u=rwX,go= ~/.ssh`
`chmod -R u=rwX,go= ~/.gnupg`

Uppercase `X` ensures that executable permission is given only to:
1. Folders
2. Files that already had executable permission anyway

`go=` ensures that `groups` and `others` permissions are set to zero.

## Algorithms

RSA - old, slow, big size
ECC (Ed25519) - new, fast, small size

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

## SSH

For securely connecting to a shell, or forwarding ports (tunneling).

My locations:
- (security)/ssh - symlinked to home/.ssh

Use simple password for SSH keys (just so that there IS password).

### Generating

Default algorithm is ED25519.
Comment can be tweaked later, but it requires changing both public and private keys.
Default comment is usually "username@hostname".
The default comment identifies WHO and FROM WHERE you are trying to connect. But it doesn't identify TO WHERE you are trying to connect.
This is fine in general, because usually you reuse the same SSH key for multiple different systems.
However, for more granular control (if you make different keys to access different systems), it makes sense tweaking the default comment a bit:

`username@hostname-target`
`ivan@laptop-domain`
`ivan@laptop-github`

Filename pattern: `id_ed25519_postfix`, e.g. on ivan@laptop machine, to connect to github: `id_ed25519_github`.
Since we do not include the "who" in the name of the key - keys should not be shared between machines (each machine generates its own).
If we ever need to centrally store these keys - these files should be renamed to specify the "who".

To generate a keypair:

- `ssh-keygen -C "comment"`

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
