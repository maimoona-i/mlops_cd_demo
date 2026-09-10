# MLOps Continuous Delivery Demo

A small Flask inference API demonstrating a Continuous Delivery (CD) pipeline using GitHub Actions, Docker, and GitHub Container Registry (GHCR).

## What this project does

- Serves a dummy ML prediction API with `/`, `/health`, and `/predict` endpoints
- Runs automated tests (pytest) on every version tag push
- Builds a Docker image and publishes it to GHCR with semantic version tags
- Deploys to a staging environment automatically after a successful build
- Runs a health/smoke test against staging
- Requires manual approval before deploying to production
- Uses immutable, versioned artifacts (build once, promote the same image through environments)

## Project structure
mlops_cd_demo/
├── app.py
├── requirements.txt
├── Dockerfile
├── VERSION
├── conftest.py
├── tests/
│ └── test_app.py
└── .github/
└── workflows/
└── cd.yml

## Running locally

```bash
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
python app.py
```

Test the health endpoint:
```bash
curl http://localhost:5000/health
```

## Running tests

```bash
pytest
```

## CD Pipeline

Pushing a semantic version tag (e.g. `v1.0.0`) triggers the pipeline:

1. **test** — installs dependencies and runs pytest
2. **build** — builds the Docker image and pushes it to `ghcr.io/<repo>` with the version tag and `latest`
3. **deploy-staging** — deploys the image to a staging server and runs a smoke test against `/health`
4. **deploy-production** — requires manual approval, then deploys the same image to production

```bash
git tag v1.0.0
git push origin v1.0.0
```

## Notes

- `deploy-staging` and `deploy-production` require `STAGING_HOST`, `STAGING_USER`, `STAGING_SSH_KEY` (and production equivalents) configured as GitHub Secrets, pointing to a live server. In this classroom exercise, no server was provided, so these jobs are expected to fail after `test` and `build` succeed.