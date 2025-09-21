# Git

## Signing commits

Refer to [keys.md](keys.md) to check the guide on how to create GPG keys.

There is no need to sign every commit - signing a tag implies everything before is good.
So we can set up only signing tags instead of signing every commit.

`git config --global commit.gpgsign true`
`git config --global tag.gpgsign true`
`git config --global user.signingkey KEY_ID!` (or `KEY_ID\!`)

KeyId MUST be appended with ! to use a specific subkey, and not the latest one.

Alternatively, to sign a specific commit:
`git commit -S`
