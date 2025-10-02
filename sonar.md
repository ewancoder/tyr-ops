# Sonar

## Setting up new monorepo on SonarCloud

1. Analyze new project.
2. Select the repository.
3. On the very bottom right, click "Setup a monorepo".
4. Choose a repository.
5. Add two (or more) projects:
  - Project key pattern: `ewancoder_aircaptain-api`, `ewancoder_aircaptain-web`, etc (the last part is monorepo module name).
  - Display name: AirCaptain Api, AirCaptain Web, etc: readable.
6. Select "previous version" as new code identifier.

Open both projects, and for each:

1. Make sure in Settings -> Administration, that we have the correct quality gate (TyR way).
2. Go to the root page of each project, choose "With GitHub Actions".
3. Copy the tokens, add them to the repository secrets.
  - `API_SONAR_TOKEN`, `WEB_SONAR_TOKEN`, `BE_FETCH_SONAR_TOKEN` (app specific) etc.
4. Deploy (analyze) them, so that the branches tab becomes active.
5. Make sure `develop` branch is included in a pattern of long-running branches.
  - pattern: `((branch|release)-.*)|(develop)` (first part is SonarCloud default).
