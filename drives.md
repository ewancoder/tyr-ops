# Drives

My drives across devices.

## Principles

- Having separate volumes (like "nda" or "work") is excessive - we can store all the data in **/mnt/data** and symlink if needed
- System & Data volumes are always full-disk encrypted
- Gaming & Media volumes are NOT encrypted (for fast gaming & media library, not sensitive content)

## Odin

My main personal PC. Used as a local media server (watching content), gaming, and workstation.

### Main drive 9100, 2Tb

- 1Gb, EFI
- 200Gb, Arch Root
- 500Gb, Arch Data
- 500Gb, Gaming Windows System
- 662 Gb - unallocated (need 560)

### Second 9100 drive, 2Tb

- 400Gb, Workstation Windows System
- 100Gb, Windows Data partition (for projects / work)
- 700Gb, Games partition - unencrypted
- 663 Gb - unallocated (need 560)

### 990 Pro drive, 4Tb

- Given to Server, for Media Downloads/storage
- (need 1110 for OP)

### 980 Pro, 2Tb

- 500Gb, Backup partition (for Arch backups, encrypted)
- Need 560Gb overprovisioning
