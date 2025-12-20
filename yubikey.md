# Yubikey

Yubikey is a hardware device that can store / act as a security key / passkey / TOTP.

## Setting up FIDO

`ykman fido access change-pin` (8-pin)

## Configuring interfaces (on/off)

Just install Android app, and use NFC, much simpler.

## Storing passkeys

Register passkeys (either passwordless passkeys, or MFA security keys) on websites & use NFC/USB interface.

## Storing SSH

Refer to keys.md.

## Signing commits / GPG

After moving a subkey to GPG, it can now be used without password (just pin) with key inserted.

Some security settings might be adjusted:

```
ykman openpgp access set-signature-policy always/once
ykman openpgp keys set-touch sig off/on/fixed/cached/cached-fixed
```

To check the signing key ID (for git configuration on new machine), use:

```
gpg --card-status
git config --local user.name xxx
git config --local user.email xxx
git config --local user.signingkey ID!
git config --local commit.gpgsign true
```
