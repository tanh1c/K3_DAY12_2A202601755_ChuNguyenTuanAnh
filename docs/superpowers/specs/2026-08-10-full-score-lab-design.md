# Full-Score Lab Design

## Goal

Complete CP1–CP5 and the CI/CD bonus using the existing lab architecture and rubric. All local tests, the Docker build, the public Railway deployment, and the GitHub Actions badge must pass.

The repository name remains unchanged at the user's request, accepting the rubric's possible 5-point deduction.

## Local implementation

Replace only the supplied stubs:

- CP1: environment-backed settings, fail-fast API key, one-line JSON logging, and lightweight liveness.
- CP3: constant-time API-key authentication, Redis sliding-window rate limiting, monthly cost guard, and the required `/ask` flow.
- CP4: Redis conversation history with trim and TTL, readiness checks, and graceful signal handling.
- CP2: multi-stage slim Docker image, non-root runtime user, health check, dynamic `PORT`, and an `agent` Compose service connected to Redis.

Preserve current routes, dependency providers, response contracts, and test-visible names. Add no new dependencies or abstractions.

## Verification

Run each checkpoint independently, then the complete local suite. Build the Docker image with Docker Desktop, verify it is below 500 MB and runs as a non-root user, start the Compose stack, and smoke-test `/health`, `/ready`, unauthenticated `/ask`, authenticated `/ask`, rate limiting, and persisted history.

## Railway deployment

Create a Railway project and Redis service. Configure `AGENT_API_KEY`, `REDIS_URL`, `RATE_LIMIT_PER_MINUTE`, `MONTHLY_BUDGET_USD`, and `LOG_LEVEL` without committing secret values. Deploy the Docker image, create a public HTTPS domain, and verify liveness, readiness, authentication, and a successful authenticated request.

Fill `DEPLOYMENT.md` with the real project details and command outputs. Add real dashboard and health screenshots. Complete all ten `exercises.md` answers using observations and measurements from the implementation and deployment.

## GitHub Actions bonus

Create one workflow triggered by pushes and pull requests to `main`:

1. `test` installs `requirements.txt` and runs local CP1–CP4 tests with CI-safe environment variables.
2. `build` builds the Docker image on a clean runner.
3. `deploy` needs both jobs, runs only for pushes to `main`, deploys to Railway using a GitHub secret, and smoke-tests the public `/health` endpoint.

Use pinned action versions. Store the Railway token in GitHub Secrets and the public URL in GitHub Variables. Add the workflow badge to README and require it to report `passing`.

## External actions and safety

The user authorizes commits, pushes to `main`, Railway project creation/deployment, and GitHub Actions secret/variable configuration. The repository must not be renamed. Never commit `.env`, generated API keys, Railway tokens, or unrelated `CLAUDE.md` content.
