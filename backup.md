# Backup

## IvanPC

On my PC, I have the following folders backing up to 2 other different drives:

- `/mnt/data/security` -> `(backup)/security`
  - Security files (keys) including `~/.ssh` and `~/.gnupg`
- `/mnt/data/tyrm/configs` -> `(backup)/tyrm-configs`
  - My media server (homelab) configs
- `/mnt/data/tyr` -> `(backup)/tyr`
  - All project development (PC) env data, including Seq
  - SEQ server with all the logs of all my pet projects (hosted on IvanPC)
- `/mnt/data/unique` -> `(backup)/unique`
  - Unique files (photos/documents/data) that should be preserved

This is the script that is responsible for this: (backup.sh)[https://github.com/ewancoder/dotfiles/blob/master/.local/bin/backup.sh]

We are also backing up configs/secrets (small data) to a separate encrypted archive shared in cloud so we can reuse it on different devices if needed.
