# GNU utils

## xargs

### -n1

`-n1` - allows passing every argument separately:

`echo a b c | xargs something` = `something a b c`
`echo a b c | xargs -n1 something` = `someting a && something b && something c`

`n2` - will pass by pairs, and etc.

`-L1` - read one line at a time, when input is multiline.

### -I

`-I` allows specifying a pattern that will be substituted with arguments, instead of just substituting at the end.

`echo a | xargs -I{} something {} somethingelse` = `something a somethingelse`

## run0

`run0` - more secure alternative to `sudo`, introduced in `systemd`

## scp

By default, scp does not preserve ownership.

- `-r` - recursive, including subdirectories
- `-p` - preserve ownership

Alternatively, archive the folder and scp it, then unarchive as needed:

- `tar -czpf /tmp/archive.tar.gz -C /data/app postgres`
  - `-c` - create archive
  - `-z` - compress with gzip
  - `-p` - preserve permissions
  - `f` - output file
  - `-C` - change to folder, so that inside archive path is `postgresql/`

Then:

- `tar -xzpf /tmp/archive.tar.gz -C /data/app`
  - `x` - extract

## Docker

You can run a command inside docker as a user:

- `docker exec -it --user 2000:2000 containername command`

## lsblk

You can visualize drives with this:

- `lsblk -f`
- `lsblk -o NAME,SIZE,LABEL` - better alternative, shows sizes of whole drives

## Zenity

Zenity is an app that can show graphical dialogues from scripts (very useful for our rdpwin script). It is a dependency of Steam, so will always be istalled.

## dd

Write an iso to usb:

- `sudo dd if=path-to-iso.iso of=/dev/sdb bs=4M status=progress oflag=sync`
