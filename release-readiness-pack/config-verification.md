# Configuration and Environment Verification

**Candidate:** Checkout API `v2.4`  
**Inspection date:** 2026-10-10  
**Scope:** `app/app.py`, `docs/architecture.md`, `k8s/configmap.yaml`, `k8s/deployment.yaml`

## Variable-by-variable audit

| Variable | Application behavior | Manifest state | Release finding |
|---|---|---|---|
| `PORT` | Defaults to `5000` for direct `app.py` execution; Gunicorn command binds fixed port `5000` | ConfigMap sets `"5000"` | Consistent today, but app variable does not configure the Gunicorn command |
| `LOG_LEVEL` | Defaults to `INFO`, upper-cased for logging | ConfigMap sets `"info"` | Acceptable; validate allowed values and avoid DEBUG in production |
| `DATABASE_URL` | If missing, app reports `warning_fallback_sqlite` | Missing from ConfigMap; no Secret reference shown | **Critical gate:** provide from an approved Secret/external secret integration, not a ConfigMap |
| `REDIS_URL` | If missing, app reports `warning_fallback_local` | Missing from ConfigMap; no Secret reference shown | **High gate:** provide from an approved Secret/external secret integration |
| `PYTHONDONTWRITEBYTECODE` | Container runtime setting | Set by Dockerfile | Present |
| `PYTHONUNBUFFERED` | Container runtime setting | Set by Dockerfile | Present |

## Observed failure mode

The `/health` handler always returns HTTP 200 with `"status": "healthy"`. If either connection-string variable is absent, it returns a warning string for that dependency, but Kubernetes readiness and liveness both use this endpoint. Consequently, a pod can be marked Ready while lacking production DB/cache configuration. A string indicating `connected` is also based only on whether the variable exists, not on a live connection test.

This is not a verified environment. Reading YAML confirms what is declared, not what is deployed or what secrets exist in a cluster.

## Required verification in target environment

- [ ] Confirm Kubernetes context, cluster, namespace, and release window with the deployment owner.
- [ ] Confirm namespace `checkout-system` exists and access is least-privilege.
- [ ] Confirm PostgreSQL service DNS, TLS policy, database/schema migration compatibility, connection limits, and credentials.
- [ ] Confirm Redis service DNS, TLS/auth policy, persistence/eviction policy, capacity, and credentials.
- [ ] Confirm `DATABASE_URL` and `REDIS_URL` are sourced using Kubernetes Secret or the approved external-secrets mechanism. Do not store raw credentials in ConfigMaps or commits.
- [ ] Verify secret key names and references using metadata only; never print secret values to CI logs or screenshots.
- [ ] Confirm expected `LOG_LEVEL`, port, resource requests/limits, probe paths, and namespace.
- [ ] Validate with the target cluster's server-side dry run and then staging smoke tests.
- [ ] Verify health/readiness reports genuine dependency status and fails readiness when mandatory dependencies are unavailable.

## Safer deployment configuration pattern

Use placeholders only in committed manifests. The actual secret object should be provisioned out of band by an authorized secret-management workflow.

Example fragment for deployment manifest, assuming the corresponding keys are already created in a Kubernetes Secret by an approved mechanism:

```yaml
env:
  - name: DATABASE_URL
    valueFrom:
      secretKeyRef:
        name: checkout-api-secrets
        key: database-url
  - name: REDIS_URL
    valueFrom:
      secretKeyRef:
        name: checkout-api-secrets
        key: redis-url
```

The exact secret name and key names must be reconciled with the platform's standard before applying. Do not commit a populated Secret manifest or a real connection string.

## Environment verification result

**Status: NOT VERIFIED / BLOCKING.** This repository does not prove that production secrets and dependencies exist, that credentials are valid, or that network policies allow access. Keep the release at No-Go until evidence from the intended non-production environment confirms the checklist above and the health-check semantics are corrected.