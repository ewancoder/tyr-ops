# Linux install setup

With my REAL script everything is automated mostly. This is the list of things we still need to do manually.

- Sign in to Mega & Dropbox
  - /mnt/data/cloud -> /cloud
  - /mnt/data/Dropbox
    - For some reason doesn't propose to sign in. Figure it out.
    - Maybe just ditch it finally. NEVER update files there - MOVE them to MEGA
- Run Steam
  - Go invisible
  - Interface
    - Untick Notify about games
    - Default location - Library
    - Untick Enable GPU accelerated rendering
  - In-Game
    - Untick Enable Steam Overlay
  - Notifications
    - Disable friend joins game
  - Add library: Storage -> /mnt/games
- Connect to Gaming WG mesh
  - asgard 10.9.0.1, odin 10.9.0.2
 - yay tlauncher-installer
  - Restore .minecraft/ from backup: ln -s /mnt/games/minecraft ~/.minecraft
  - Select ftb modpack
- Pavucontros - set proper input/output default devices
- Telegram -> connect
  - Notifications -> do not allow sound
- Discord -> connect
  - Disable automatic streamer mode (?? didn't find where)
- Run through XPlane guide to set up everything that's required [FRESH] tag
- Pair to the TV (lgtvcmd)
  - TODO: Backup the database and restore it on install
- VS Code run
  - Install VIM extension

