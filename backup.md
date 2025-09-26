# Backup

## IvanPC

On my PC, I have the following folders backing up to 2 other different drives:

- `/mnt/data/security` -> `(backup)/security`
  - Security files (keys) including `~/.ssh` and `~/.gnupg`
- `/mnt/data/media/configs` -> `(backup)/media-configs`
  - My media server (homelab) configs
- `/mnt/data/seq` -> `(backup)/seq`
  - SEQ server with all the logs of all my pet projects (hosted on IvanPC)
- `/mnt/data/unique` -> `(backup)/unique`
  - Unique files (photos/documents/data) that should be preserved

This is the script that is responsible for this: (backup.sh)[https://github.com/ewancoder/dotfiles/blob/master/.local/bin/backup.sh]
