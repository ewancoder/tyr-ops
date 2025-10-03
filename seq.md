# Seq

Setting up seq. First login with default login/password (admin username).

- Go to Settings, API - check require authentication for HTTP/S ingestion, check that logs stopped coming.
- Go to Settings, Security, change authentication provider: set username/password for the first administrator.
- Go to Users tab, check the user.
- Sign out, sign in, check that it works.
- Data, Storage, Manually delete events, delete all.
- Theme - Dark

## Set up API keys

Seq logging sets up the following data fields in its API Key application properties:

> Application name: unique name for the application just for the logging to quickly identify the app. Since it's easier to type everything lowercase - we get app name from docker compose files: aircaptain, nitrotype-tracker, foulbot.

- `Application=` (aircaptain, foulbot)
- `App = the same ^`
- `Environment=Production/Development/etc` - camel case large name.
- `Env = the same ^`
- `Service=tyr-prod-aircaptain_api` (full docker stack name)
  - For legacy services - we still name them with the new pattern (not matching current docker name), so that logs are compatible with future changes: `tyr-prod-kartman_api`

The name of the API key itself is a docker service name because it uniquely identifies separate API keys.
