# Dependency and Supply-Chain Audit

**Inspection date:** 2026-10-10  
**Candidate:** `v2.4`

## Declared runtime alignment

Architecture contract in `docs/architecture.md` specifies Python 3.11, Flask 3.0.2, Gunicorn 22.0.0, PostgreSQL through `DATABASE_URL`, and Redis through `REDIS_URL`.

| Component | Declared version / reference | Assessment |
|---|---|---|
| Python | `3.11` in CI and `python:3.11-slim` image | Major/minor aligns with architecture; patch-level base image is not pinned |
| Flask | `3.0.2` in `app/requirements.txt` | Exact top-level pin present |
| Gunicorn | `22.0.0` in `app/requirements.txt` | Exact top-level pin present |
| pytest | `8.0.2` in `app/requirements.txt` | Exact pin present; test tooling is installed by same requirements file as runtime |
| Transitive Python dependencies | Not individually locked with hashes | Resolved graph may drift; no lock with hashes committed |
| Container base | `python:3.11-slim` | Mutable tag; no immutable digest recorded |
| GitHub Actions | `actions/checkout@v4`, `actions/setup-python@v5`, Docker actions by major tag | Major version references can move; pin to reviewed full commit SHAs in hardened CI |
| Application image | `kalvium/checkout-api:v2.4` with `IfNotPresent` | Mutable tag, no digest; node cache can cause different nodes to run different bytes |

## Security finding

The repository contains exact pins for three direct Python packages, which is better than unconstrained versions. However, exact direct pins alone do not establish that the full dependency graph is safe or reproducible. This audit did **not** execute pip-audit, OS package scanning, image scanning, or check a live vulnerability database. Therefore, it makes no claim of “zero known vulnerabilities.”

## Commands to run and capture

Use the same Python 3.11 environment intended for CI:

```bash
python --version
python -m pip --version
python -m pip install -r app/requirements.txt
python -m pip freeze --all | tee release-readiness-pack/evidence/pip-freeze.txt
python -m pip install pip-audit
python -m pip_audit -r app/requirements.txt
```

For a clean, reproducible environment, generate a hash-locked requirements file using an approved tool and review the resulting resolved versions before committing it. Keep test-only dependencies in a development/test requirements group if the build process supports it.

Container checks (requires Docker and a trusted scanner installed):

```bash
docker build --pull -t checkout-api:v2.4-review ./app
docker image inspect checkout-api:v2.4-review --format '{{json .RepoDigests}}'
```

Run the organization's approved container vulnerability scanner against the locally built image. Record scanner name/version, database update time, image digest, findings by severity, and exceptions with owners. A locally tagged image may not have a repository digest; publish to an approved registry before capturing the immutable registry digest.

## Required hardening before Go

1. Resolve and lock transitive dependencies, ideally with hashes; separate runtime dependencies from test-only tooling.
2. Run Python package audit and an OS/container image scan against current vulnerability data; remediate or formally accept findings.
3. Pin `python:3.11-slim` to an approved digest and schedule reviewed base-image refreshes.
4. Pin GitHub Actions to reviewed full commit SHAs and use minimal `GITHUB_TOKEN` permissions.
5. Publish the built image once, record its immutable digest, and deploy the digest rather than `kalvium/checkout-api:v2.4`.
6. Verify image provenance/signature and registry access policy if the platform supports them.
7. Re-run CI and vulnerability checks after any dependency or image update.

**Result: PARTIAL.** Direct top-level dependency versions are pinned, but dependency vulnerability status and immutable artifact identity are unverified. This remains a release gate.