# WireGuard setup for local network (PC)

1. Open port on router
2. Add ip forward:

```
echo "net.ipv4.ip_forward=1" > /etc/sysctl.d/99-wireguard.conf
sysctl --system
```

3. UFW rules

```
ufw allow ${WG_PORT}/udp
ufw allow in on $WG_IFACE
ufw allow out on $WG_IFACE
ufw route allow in on $WG_IFACE out on $WG_IFACE
ufw disable
ufw enable
```
