# Networking

- `nbtscan 192.168.1.0/24` - will show you which PCs with their domain names have which IP addresses, on the local network

## Setting up static local IP address

Using `iwd`/`systemd-networkd` it's pretty simple. Create a file `/etc/systemd/network/20-wifi.network`:

```
[Match]
Name=wlan0

[Network]
Address=192.168.0.185/24
Gateway=192.168.0.1
DNS=1.1.1.1
DNS=8.8.8.8
```
