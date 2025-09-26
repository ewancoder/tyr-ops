# Repositories

For TypingRealm (TyR) project repos, I have created a separate organization:

- `ewancoder-tyr`

In future we might rename it to `typingrealm`.

So all repositories related to TyR will eventually migrate to `https://github.com/ewancoder-tyr`.

This has a disadvantage of not having them in my public profile, but advantage of having them all nicely scoped together and having an ability to manage organization-wide secrets for github actions.

## Organization

We created a new organization `ewancoder-tyr`.

Now we need to go to the settings and do the following:

- `Packages` tab - set package creation to `Public` by default.

## Adding a repository

> Make sure it's **public**. It creates them private by default. Private repos won't be able to use organization secrets in a free plan.

## Setting up GIST secret

TyR projects depend on `GIST_SECRET` secret - PAT for updating GIST secrets, for badges creation.

1. Click on profile icon -> Settings.
2. Developer settings (the very bottom).
3. Personal access tokens
4. Classic tokens - generate new classic token
  - Token note: GIST Badges
  - No expiration
  - Just the GIST permission
5. Add as TyR organization action secret `GIST_SECRET`

## Setting up Sonar secrets

See (sonar.md)[sonar.md] for a guide on how to set up Sonar secrets.

> NOTE that SONAR secrets should still be created **PER REPOSITORY**, because they are separate for each project.

## Setting up package permissions

When moving between **personal** repos, new repos cannot publish packages to ghcr.io because packages are tied to a single repo. To fix this:

1. Click on profile pic, go to Profile.
2. Packages tab.
3. Select package(s) in question.
4. Open package settings (on the right).
5. Add a repository there.

However, when moving from personal repository to an organization - the whole namespace (org) changes, so we need completely new packages. So this is unnecessary - pipelines will create completely separate packages. We only need to adjust deployment files to reference the new user: `ewancoder-tyr` instead of `ewancoder`.

## Deployment secrets

> This part will be moved to deployment guide later.

Set up the following secrets on the Organization level for TyR deployment:

- `SSH_HOST`
- `SSH_PORT`
- `SSH_USERNAME`
- `SSH_KEY`
- `SSH_KEY_PASSPHRASE`
