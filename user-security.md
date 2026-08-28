# Adding secure user to system

sudo groupadd --gid 5000 egor

sudo useradd \
  --uid 5000 \
  --gid 5000 \
  --no-user-group \
  --create-home \
  --home-dir /home/egor \
  --shell /bin/bash \
  --expiredate 2026-12-31 \
  --comment "Egor" \
  egor

passwd -S egor (disable password)

btrfs quota enable /home
btrfs subvolume create /home/egor
btrfs qgroup limit 5G /home/egor
