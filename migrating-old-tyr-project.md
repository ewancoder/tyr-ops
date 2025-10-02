# Migrating an old TyR project to new infra / naming conventions

- Create the project on `ewancoder-tyr` (typingrealm) org
- `git remote add origin git@github:ewancoder-tyr/$APP.git`
- Copy `.github` folder, change project-specific env vars in `deploy.yml`
- Tweak `.env.$ENV` files & `swarm-compose.yml`
  - Do not forget to update `ghcr:ewancoder` to `ghcr:ewancoder-tyr` for tyr packages
- Upgrade `HostExtensions.cs` to the latest versions (supporting Swarm secrets reading)
  - And update `Program.cs` if needed
  - Also we need to update csproj, and Packages.props files, cause we added more deps.
- Create needed infra on cluster
- After everything is working - adjust `README.md` to show all the badges

Short example of upgrading infra for a new project:

```
cd /data/tyr
mkdir -p {dev,prod}/overlab/secrets
cp $OLD_SECRET_LOCATION prod/overlab/secrets/env
cp $OLD_SECRET_LOCATION dev/overlab/secrets/env
# Adjust any outdated info in the env files (like Seq API keys, and removing global secrets & migration secret)
# ^ Actually, copy migration secret to prod/overlab/secrets.env (needed for migrations). (DATABASE_URI)
```

Make sure secrets are tyr:tyr 600.

> Database name pattern: $ENV-$APP (prod-overlab). So we can store databases from different envs on the same server if need be.

- Set up repository-level secrets:
  - `API_SONAR_TOKEN`
  - `WEB_SONAR_TOKEN`
  - `BE_FETCH_SONAR_TOKEN` - example for a project with `Be_Fetch` backend (nitrotype-tracker)

- Create folders on both prod-infra and dev-infra machines:
  - app/{postgres,cache}
