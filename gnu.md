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

## sed

Delete first 10 lines from a file:

`sed -i '1,10d' file.txt`

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

## udisksctl

Can be used instead of mount/umount to mount partitions/usb drives by user (without sudo).

- `udisksctl lock/unlock -b /dev/device` - lock/unlock if it's a LUKS-encrypted partition
- `udisksctl mount/unmount -b /dev/device (or /dev/dm-X in case of luks)` - mount partition

## Sleep setup

- `cat /sys/power/mem_sleep` shows with `[]` bracked which one is currently active
  - `s2idle` - buggy windows low-energy mode, usually on laptops
  - `deep` - true S3 level sleep, what we need

If it's `s2idle`, we need to edit GRUB parameters to pass to kernel to use `S3`. Edit `/etc/default/grub` and regenerate config:

```
GRUB_CMDLINE_LINUX_DEFAULT="... mem_sleep_default=deep"
```

> BUT it bricks the laptop lol. We need to hold power button for a whole minute to do a hard-hard-reset then. Resume still didn't work from the box.

I guess it's not possible to suspend with Nvidia always on mode in the laptop. I need to either use integrated graphics, or move to hibernate / shutdown.

## ASUS laptop tweaks

> TODO: Move this section to a separate file.

Check current status:

```
supergfx -g
lspci -k
grep -A 3 "VGA" | lsmod | grep nvidia` - check current status
```

- `supergfxctl -m Integrated` - switch NVIDIA completely OFF (run everything off integrated graphics)
- `supergfxctl -m Hybrid`
- `supergfxctl -m Discrete`

> Need to reboot after.

> `supergfxctl` hangs and doesn't work when in Integrated mode cause no Nvidia GPU. So if you switch to Integrated mode - it's impossible to switch back (unless passing kernel params or using windows/asus/g-helper software)

### Checking battery

- `/sys/class/power_supply/BAT0/capacity` - 73
- `/sys/class/power_supply/BAT0/status` - discharging

### Sleep

Basically we cannot achieve reliable sleep with neither Discrete nor Integrated modes, we need to use Hybrid mode. In Discrete mode we cannot resume from sleep, in Integrated mode we cannot reach sleep. Even reboot doesn't work in Integrated mode.

We need to use Windows `g-helper` to change modes instead of Linux, more reliable for this laptop.
