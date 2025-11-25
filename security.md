# Security

## Device security

### Secure boot

Secure boot prevents boot from unsigned bootloaders.

Secure boot chip contains a DB (signature database) with trusted public keys for bootloaders.

Let's say Microsoft owns private key `A_Prt` with a public key pair `A_Pub`. Then it signs install CDs / their systems with this private `A_Prt` key.

Manufacturers of all motherboards agree to include `A_Pub` key in the database of trusted public keys in their secure boot chips. This is why all motherboards trust Windows Install ISO (and installed Windows OS / bootloader).

If you need to install Arch Linux - the Arch Linux live CD is not signed with the trusted key. The only way to load it without disabling secure boot - is to sign it yourself with some custom personal generated key, and enroll its public pair into your own BIOS / secure boot.

So, in general the flow is this:

1. Generate a key pair that will be used for signing live CDs / OSs.
2. Enroll its public pair on all secure boots of all your devices.
3. Sign any live CDs (or installed GRUBs) with this key - it will be trusted by your devices now.

Generate a key (or use your pre-generated):

```
openssl req -new -x509 -newkey rsa:4096 -keyout mok.key -out mok.crt -nodes -days 3650 -subj "/CN=Boot/"
```

- `req` - certificate signing request (CSR)
- `-new` - generate new certificate
- `-x509` - use x509
- `-newkey rsa:4096` - create a new private key as well, RSA 4096-bit strength
- `-keyout mok.key` - file for a private key
- `-out mok.crt` - file for the certificate (MOK = machine owner key)
- `-nodes` - stands for no DES - don't encrypt private key with a passphrase
  - This is important for booting, because bootloaders cannot ask for a password to read the key
- `-days 3650` - 10 years expiration
- `-subj` - subject name, the reason for the key

> Consider creating separate keys for separate goals, and naming the `-subj` accordingly.

Sign GRUB binary:

```
sbsign --key mok.key --cert mok.crt /boot/EFI/<distro>/grubx64.efi --output /boot/EFI/<distro>/grubx64.efi.signed
```

Convert CER to DER (for mok enrollment to firmware):

```
openssl x509 -in MOK.crt -out MOK.der -outform DER
```

Enroll to firmware (can be done from OS):

```
sudo mokutil --import mok.der
reboot
```

> It will ask for "input password" - it's a temporary password to make sure nobody tempers with the process.

Turn on secure boot, reboot. Now, in theory, blue screen of MOK enrollment should happen. However it looks like we run into chicken and egg problem here: secure boot sees that first EFI entry is not secure and does not let us enroll the key.

Solutions to this problem:

1. Temporarily use Microsoft-signed shim.
2. Enroll the key manually via the BIOS (from USB drive). I've used this way.

WIPWIP

Well lol this didn't work as well. bios didn't let enrolling public keys for some reason. but it did have an option to enroll whole EFI so i enrolled grub efi and managed to have the next error:

kern/efi/sb.c:shim_lock_verifier_init:175:prohibited by secure boot policy lololol

it looks like i need to create gnupg key pair and sign grub FONT lol.

also good idea to generate my own machine PK for secure boot, do not use default / from manufacturer, cause it might be weak in general i guess? if it's shared between machines at least but arch wiki also mentions it with some source link

! BUT some firmware might stop working like gpu of a laptop because many of them use some kind of microsoft issued keys that are required
!!! and it might be even impossible to get back into the uefi menu if keys are altered so better use those that are provided by default

### WIP

TODO: firewalls and antivirus

##### UKI GUIDE

1. /etc/mkinitcpio.d/linux.preset
- `uncomment default_uki (and make sure it's pointing to the right folder, !!! redo this to remember) - and default_image can be commented, we don't need to generate initramfs separate file anymore`

- add 'microcode' hook in mkinitcpio hook (can be added at the very end) to prepend microcode when generating image (amd-ucode)
- OH.. microcode is already there by default lol that's nice

to verify the uki: lsinitcpio EFI | less - shows list of files
we also need sd-encrypt before filesystem hook, for encrcyption (and make sure sd-vconsole and keyboard hooks are present)

# this checks the root partition / cmdline arguments:
 strings /boot/EFI/Linux/arch-linux.efi | grep -i root 



# fwupd

pacman -S fwupd
fwupdmgr refresh --force # updates database of latest updates
fwupdgmr get-devices # list devices for firmware updates
fwupdmgr get-updates # check available updates
fwupdmgr update # apply update

> lvfs is Linux Vendor Firmware Service, vendors can make firmware for use by fwupdmgr

ok good way to do this:
- load default firmware keys
- enroll EFIs of systemd + linux to load into linux
- now backup all dbx etc
- then check fwupdmgr get-updates, and update dbx/uefi/ca etc fwupdmgr update
  - !!! this also FINALLY updates it from 2011 to 2023, hopefully it updates the DEFAULT database, so I can re-defaultify it in bios and it'll still remain 2023
  - Also contains latest dbx 2016 -> 2023 lol
- backup current keys:
  - sbctl status: microsoft, builtin-db, builtin-db, builtin-KEK, builtin-PK
  - for var in PK KEK db dbx; do efi-readvar -v $var -o sb/old_${var}.esl; done # needs packaefitools package
- generate merged db:
  - cert-to-efi-sig-list /var/lib/sbctl/keys/db/db.pem db.esl
  - cat old_db.esl db.esl > merged_db.esl
- efi-updatevar -e -k /var/lib/sbctl/keys/KEK/KEK.key -f sb/old_dbx.esl dbx
- efi-updatevar -e -k /var/lib/sbctl/keys/KEK/KEK.key -f sb/merged_db.esl db
- chattr -i /sys/firmware/efi/efivars/db-*
- sbctl enroll-keys --partial PK,KEK -fm

