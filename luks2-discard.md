# LUKS2 and Trim

For LUKS2 drives to be able to Trim - we need to do this:

```
cryptsetup refresh --allow-discards --persistent root
cryptsetup refresh --allow-discards --persistent data # Etc.
```
