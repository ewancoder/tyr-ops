# GNU utils

## xargs

### -n1

`-n1` - allows passing every argument separately:

`echo a b c | xargs something` = `something a b c`
`echo a b c | xargs -n1 something` = `someting a && something b && something c`

`n2` - will pass by pairs, and etc.

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
