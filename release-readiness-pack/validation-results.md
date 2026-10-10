# Validation Results: Checkout API v2.4

**Assessment date:** 2026-10-10  
**Candidate:** `v2.4`  
**Baseline commit:** `ebe39e6cd8a2bb4c18e9893e7c0d4344ca925227` (`main` at assessment)  
**Evidence policy:** This record distinguishes repository inspection from commands actually executed. No fabricated local output or CI run is presented as evidence.

## Summary

| Check | Result | Evidence / follow-up |
|---|---|---|
| Source and test inspection | Inspected | `app/app.py`, `app/test_app.py` |
| CI workflow definition | Inspected | `.github/workflows/ci.yml` defines Python 3.11, syntax compilation, pytest, and Docker build |
| CI execution | **Not observed** | GitHub Actions runs endpoint returned `total_count: 0` at assessment; rerun after pushing this branch |
| Unit tests | **Not executed in this assessment** | Run the commands below locally or rely on the new PR's Actions run, and capture actual output |
| Python syntax check | **Not executed in this assessment** | Command documented below |
| Container build | **Not executed in this assessment** | Docker availability and actual image build remain unverified |
| Kubernetes manifest validation | **Not executed in this assessment** | No cluster / server-side dry-run evidence available |
| Production dependency connectivity | **Not verified** | No production credentials or reachable DB/Redis endpoints supplied |
| Runtime health semantics | **Fail / design gap** | `/health` always returns HTTP 200 and `status: healthy`, even when DB and Redis settings are missing |

## Repository evidence inspected

- Workflow: [CI Pipeline](../.github/workflows/ci.yml). It installs `app/requirements.txt`, runs `python -m py_compile app/app.py app/test_app.py`, runs `pytest app/test_app.py`, and builds the Docker image without pushing.
- Test source: [`app/test_app.py`](../app/test_app.py) contains `test_index` and `test_health`. The health test currently verifies only response structure, not production dependency availability or failure semantics.
- Container definition: [`app/Dockerfile`](../app/Dockerfile) uses `python:3.11-slim` and runs Gunicorn.
- Manifest health probes: [`k8s/deployment.yaml`](../k8s/deployment.yaml) uses `/health` for both readiness and liveness.
- No Actions workflow runs were returned by the repository Actions API at inspection time. This is absence of observed run evidence, not proof that code would fail or pass.

## Reproducible validation commands

Run from the repository root:

```bash
python3 --version
python3 -m venv .venv
. .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r app/requirements.txt
python -m py_compile app/app.py app/test_app.py
python -m pytest -v app/test_app.py
docker build --pull -t checkout-api:v2.4-review ./app
```

If Docker is unavailable, mark the container build as not run rather than passing it by assumption. Save the terminal output and environment/tool versions to `evidence/` before release review.

Kubernetes validation, from a controlled cluster context:

```bash
kubectl config current-context
kubectl apply --dry-run=server -f k8s/configmap.yaml
kubectl apply --dry-run=server -f k8s/deployment.yaml
kubectl apply --dry-run=server -f k8s/service.yaml
```

Do not run server-side commands against production merely to satisfy this assignment. Use an approved test cluster and record its context.

## Required acceptance criteria

- [ ] PR Actions run is complete and green for the exact candidate commit.
- [ ] Unit test output is saved with timestamp and commit SHA.
- [ ] Docker build passes; resulting image digest is recorded.
- [ ] Manifest schema/admission validation passes against the intended Kubernetes version.
- [ ] Staging smoke test confirms `/` returns the expected candidate version.
- [ ] Health/readiness checks fail closed or clearly report degraded status when required dependencies are missing/unreachable.
- [ ] Staging test uses real non-production PostgreSQL and Redis services with valid secret injection.
- [ ] Rollback rehearsal succeeds in staging and restoration is verified.

**Current validation conclusion:** Evidence is insufficient for release approval. The workflow definition and source are inspected, but execution evidence must be captured by an actual run.