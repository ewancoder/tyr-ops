# Security

## Device security

### Secure boot

Secure boot chip (in the motherboard) contains a list of allowed (and prohibited) signatures to not allow any bootloaders that are not signed by a trusted signature to load.

In order to load any software (Windows OS, Windows installer, Arch live CD) you need it signed with a signature that your motherboard trusts.

By default, motherboard manufacturers trust a specific signature that is being used to sign all Windows OS (and installers), so Windows works out of the box. However, in order to be able to boot Linux live CDs or systems - you need to sign them with a private key, a public pair for which is stored in your motherboard (e.g. Microsoft private key).

There are two solutions to this problem.

1. Use Microsoft-provided "shims" to sideload your bootloader. Downsides being - relying on Microsoft, and some (maybe) security concerns due to shims in turn trusting your unsigned bootloaders.
2. Enroll your own private keys to your secure boot firmware (motherboard), and sign your bootloaders using these keys, so your motherboard natively trusts your software.

I am using a second approach.

#### Setup guide

> Prerequisites: this is all mostly pointless if third party can enter a BIOS and disable secure boot, so setting up a BIOS password is recommended. Just make sure you will NEVER forget it (as I did for an older laptop).

1. First we want to make sure that you have the latest and greatest database of signatures (DB - allowed signatures, DBx - forbidden signatures) in your firmware.

Load into BIOS, turn on Secure boot, and switch it into "User" mode (by deleting factory keys, if there is no dedicated option).

Now update your motherboard firmware using `fwupdmgr`:

```
pacman -S fwupd
fwupdmgr refresh
fwupdmgr get-updates
fwupdmgr update
reboot
```

Reboot, load BIOS and make sure keys are present there. There should be 6-10 DB keys and 400+ DBx keys. The BIOS should also be switched into "User" mode now.

Enroll your UKI (and bootloaders) EFI images to your bios, selecting an option via BIOS menu to enroll into it (trust a EFI image, or import signature from EFI image).

Now boot into your linux. You will be able to do this if you correctly enrolled your EFIs to your secure boot. This was just a temporary measure to boot into the system to back up our manufacturer/Windows keys to reuse them later.

2. Back up latest and greatest firmware/Microsoft keys.

```
pacman -S efitools
mkdir sb && for var in PK KEK db dbx; do efi-readvar -v $var -o sb/${var}.esl; done
```

This will read your keys from efivars and save them to the `sb` folder.

3. Install sbctl and generate keys (or restore from backup if you have it on other machines).

```
pacman -S sbctl
sbctl create-keys # to create new keys
```

Or restore the keys at `/var/lib/sbctl/keys` folder if you already have them.

4. Merge the databases

Now we are merging the sbctl generated keys with microsoft/vendor keys:

```
cert-to-efi-sig-list /var/lib/sbctl/keys/db/db.pem mydb.esl
cat sb/db.esl mydb.esl > merged_db.esl
```

5. Enroll all the keys to Secure Boot

Reboot and switch Secure Boot into "Setup" mode, then:

```
efi-updatevar -e -k /var/lib/sbctl/keys/KEK/KEK.key -f sb/dbx.esl dbx
efi-updatevar -e -k /var/lib/sbctl/keys/KEK/KEK.key -f merged_db.esl db
chattr -i /sys/firmware/efi/efivars/db-* # We need this to make keys writable again for final KEK/PK writes
sbctl enroll-keys --partial PK KEK -fm
```

For some reason, it enrolls only PK (no KEK) but it is fine. KEK is used for re-inrollment of new keys but we own our hardware so we can just re-setup everything if needed. The important thing is that DB and DBx are present with both Microsoft, Vendor and our own personal (sbctl generated) keys.

6. Sign your EFIs with sbctl

Sign all your EFIs (bootloader, UKI) with sbctl:

```
sbctl sign -s path-to-efi # -s saves the binary path so you can easily re-sign it later, but it's not mandatory
```
