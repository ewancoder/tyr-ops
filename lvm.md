# LVM

Lvm allows us transparently joining multiple drives into a single bigger drive.

```
pvcreate /dev/sda
vgcreate media-vg /dev/sda
lvcreate -l 100%FREE -n media-lv media-vg
# or -L 20T
mkfs.ext4 /dev/media-vg/media-lv
```

Then just mount it / add to fstab:

```
mount /dev/media-vg/media-lv /folder-to-mount-to
fstab: /dev/media-vg/media-lv /folder ext4 defaults 0 2
```

Use `0 2` for ext4 fs, not `0 0`, 0 disabled `fsck` (not needed for btrfs).

!!! install lvm2 package on arch
