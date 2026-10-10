# Rollback Plan: Checkout API

**Candidate:** `v2.4`  
**Known-good version in repository release history:** `v2.3` (marked “Deployed (Prod - STABLE)” in `docs/architecture.md`)  
**Status:** Procedure drafted; rehearsal **not performed**. Do not label this a tested rollback until a staging rehearsal and verification evidence are attached.

## Trigger conditions

The Release Owner or SRE/Operational Lead must stop rollout and initiate rollback if any of the following occurs during the deployment window:

- checkout error rate or latency exceeds the agreed SLO / alert threshold;
- readiness failure, crash loops, or elevated 5xx persist beyond the agreed observation window;
- database/cache connectivity is missing or degraded;
- payment or checkout smoke tests fail;
- severe unexpected logs, data-integrity concerns, or customer-impacting regression;
- monitoring is unavailable and release health cannot be established.

Record the incident/ticket, approver, UTC/local timestamp, triggering metric, and current image digest.

## Preconditions

- [ ] Confirm the cluster context is the intended **production** context only for a genuinely authorized rollback.
- [ ] Confirm `checkout-system` namespace and deployment name `checkout-api`.
- [ ] Retrieve the immutable digest of the actual previously deployed `v2.3` image from the trusted registry/release record. The repository history names `v2.3` as stable but does **not** provide an immutable image digest here.
- [ ] Confirm v2.3 is compatible with current database schema and any migrations. If migrations are not backward-compatible, halt and follow the database recovery runbook.
- [ ] Confirm the person executing rollback has permissions and the on-call lead is present.
- [ ] Capture current deployment image, revision, ReplicaSet, pod state, and event snapshot for audit.

## Preferred rollback: restore the last known-good image

Use the real immutable digest resolved from the registry. Do **not** copy the placeholder literally.

```bash
kubectl config current-context
kubectl -n checkout-system get deployment checkout-api -o wide
kubectl -n checkout-system get deployment checkout-api -o jsonpath='{.spec.template.spec.containers[?(@.name=="checkout-api")].image}{"\n"}'
kubectl -n checkout-system rollout history deployment/checkout-api
# Substitute the verified registry path and immutable v2.3 digest:
kubectl -n checkout-system set image deployment/checkout-api checkout-api=REGISTRY/checkout-api@sha256:VERIFIED_V2_3_DIGEST
kubectl -n checkout-system rollout status deployment/checkout-api --timeout=180s
kubectl -n checkout-system get pods -l app=checkout-api -o wide
kubectl -n checkout-system get events --sort-by=.lastTimestamp
```

Only execute the image change after replacing the placeholder with the verified v2.3 digest and confirming the context. If deployment history and registry metadata show an audited known-good prior ReplicaSet, `kubectl rollout undo deployment/checkout-api` is an alternative, but verify the resulting image and revision afterward. Do not rely on undo blindly if history has been pruned or deployment automation may immediately reapply v2.4.

## Post-rollback verification

1. Confirm all expected replicas are available and rollout has completed.
2. Confirm the running image matches the recorded known-good v2.3 digest.
3. Run smoke tests through the service endpoint: version response, checkout happy-path test, payment integration in sandbox, and relevant error-path tests.
4. Confirm DB/Redis connectivity and health/readiness signals using genuine dependency checks, not only HTTP 200.
5. Observe error rate, latency, saturation, and application logs for the agreed observation window.
6. Obtain incident/on-call confirmation, communicate customer-impact status, and update the release record.
7. Preserve failed candidate digest, events, logs, timestamps, and the reason for rollback for investigation.

## Database and state caveat

Rolling back the application binary does not reverse database migrations or external side effects. No migration artifacts or data recovery procedure were provided in the inspected repository. The release must not proceed until schema compatibility and data recovery ownership are confirmed.

## Staging rehearsal record

**Current result: NOT RUN.** Rehearse the complete procedure in staging with a previous known-good digest. Attach command output and timestamps to `evidence/rollback-rehearsal.md`; include pre-rollback and post-rollback version, smoke-test results, and the responsible operator. Never fabricate a “passed” result.

**Rollback gate:** No-Go until immutable v2.3 artifact and schema compatibility are verified and the rehearsal passes.