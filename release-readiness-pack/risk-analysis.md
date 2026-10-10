# Risk Analysis: Checkout API v2.4

**Assessment date:** 2026-10-10  
**Scoring:** Severity is impact if the risk materializes. Likelihood is a qualitative estimate from evidence currently available. “Owner” is an accountable role placeholder; the release coordinator must assign a named person before deployment.

| ID | Risk / evidence | Severity | Likelihood | Mitigation / exit criteria | Accountable owner | Release status |
|---|---|---|---|---|---|---|
| R1 | **Database and cache configuration can silently fall back.** ConfigMap omits `DATABASE_URL` / `REDIS_URL`; `/health` still reports healthy/200. | High | High | Inject values through approved Secrets; correct health/readiness semantics; prove in staging that missing/unreachable mandatory dependencies prevent readiness and trigger alerts. | Application Owner + SRE Lead | Open, blocker |
| R2 | **Rollback artifact is not immutable.** Docs identify v2.3 as stable, but no registry digest is captured; deployment uses mutable `kalvium/checkout-api:v2.4`. | High | Medium | Resolve trusted v2.3 digest, confirm schema compatibility, deploy by digest, rehearse rollback in staging and attach evidence. | Release Owner + Build/Release Engineer | Open, blocker |
| R3 | **Health checks can give false confidence.** Health endpoint checks only whether URLs are present, not actual connectivity; always emits healthy/200. | High | High | Implement meaningful dependency checks with safe timeouts; separate liveness from readiness; test healthy/unhealthy dependency scenarios and ensure no retry storm. | Application Owner | Open, blocker |
| R4 | **Dependency / base-image vulnerability exposure is unknown.** Top-level pins exist but transitive graph, OS packages, and current CVE status are not audited; base tag is mutable. | High | Medium | Generate reviewed lock with hashes; execute package and image scanners against current databases; remediate or obtain documented security exception; record build digest. | Security/Platform Engineer | Open, blocker until scan |
| R5 | **Runtime artifact reproducibility is weak.** CI Action refs use major tags and app container image is mutable. | Medium | Medium | Pin actions to reviewed SHAs; deploy image digest; keep provenance and build metadata. | Platform Engineer | Open |
| R6 | **Rollback may not restore data compatibility.** No migration history/reversibility evidence is supplied; app rollback alone does not undo data writes/schema changes. | High | Medium | Review migration diff, backward compatibility and recovery path; rehearse against staging copy; get DB owner approval. | Database Owner | Open, blocker |
| R7 | **No operational names or approval evidence.** The repository defines no named release/on-call operators or escalation contact. | High | Medium | Assign names, reachable contacts, coverage for full window, explicit approving reviewer, and escalation chain. | Release Manager | Open, blocker |
| R8 | **No observed CI run/evidence at assessment.** Actions API returned zero workflow runs; command execution and cluster validation are not evidenced here. | High | Medium | Trigger PR Actions; require passing checks; record build/test/scanner outputs and staging smoke test for exact commit SHA. | CI Owner + Release Owner | Open, blocker |
| R9 | **Cluster/context and manifest compatibility are unverified.** No server-side dry run, namespace check, network policy, or staging rollout output supplied. | Medium | Medium | Verify approved cluster/version; server-side dry run; review RBAC/network policy/resources/probes; stage rollout and observe. | SRE Lead | Open |
| R10 | **Main branch protection is disabled.** Branch metadata reports protection disabled and required checks off. | Medium | Medium | Enable required PR review and green CI checks before relying on governance for production repository workflows. | Repository Admin | Open |

## Prioritization

The worst open risk is **High**. Risks R1, R2, R3, R4, R6, R7, and R8 are explicit release blockers. Medium-severity items also need tracked owners and closure; a low-risk label would not be justified by current evidence.

## Stop/go rules

- **No-Go:** any High risk has no validated mitigation, required CI evidence is missing, rollback target is not proven, required secrets/dependencies are unverified, or a named operator/approver is absent.
- **Conditional review:** may begin after evidence is attached, but this is not deployment authorization.
- **Go:** only after all blockers are closed or formally accepted by an authorized risk owner, validation and rollback rehearsal pass, operational owners are named and reachable, and the release approver signs the final report.

**Ownership caveat:** roles in this report are responsibilities, not real-person assignments. Enter actual names and contact routes in `ownership.md`; never imply a person accepted responsibility before they have.