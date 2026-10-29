# Resizing FS

1. dump the partitions just in case: `sfdisk --dump /dev/nvme0n1 > ~/nvme0n1-before.sfdisk`
2. Increase size: `echo ',+200GiB' | sfdisk -N 1 (partition number) --no-reread /dev/nvme0n1`
3. partx -u /dev/nvme0n1 # Tell the kernel the new size.
4. cryptsetup resize backup (luks mapper name) # Tell LUKS to grow the partition.
5. btrfs filesystem resize max /mnt/backup (mount point) # Finally resize to max btrfs filesystem.
