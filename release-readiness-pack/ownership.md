# Release Ownership and Escalation

**Release:** Checkout API v2.4  
**Proposed window:** To be scheduled after release gates pass  
**Current approval state:** No-Go  
**Owner assignment state:** Roles defined; named personnel and acknowledgements remain outstanding.

## Required named assignments

| Role | Responsibility | Required record before deployment | Current assignment |
|---|---|---|---|
| Release Owner / Coordinator | Owns checklist, go/no-go recommendation, release window, communication, and final record | Full name, team, contact, timezone, explicit acknowledgement | **Unassigned** |
| Application Owner | Confirms code behavior, dependency semantics, smoke tests, and regression triage | Full name, contact, test evidence | **Unassigned** |
| SRE / Operational Lead | Verifies deployment mechanics, dashboards, alarms, capacity, probes, and rollback execution | Full name, contact, cluster context authorization | **Unassigned** |
| Database Owner | Confirms schema compatibility and data recovery implications for rollback | Full name, contact, migration approval | **Unassigned** |
| Security / Platform Reviewer | Signs off on dependency and image scan findings and exceptions | Full name, contact, scan report | **Unassigned** |
| Primary On-Call | Monitors rollout and receives page/escalation during the entire window | Name, current paging route, acknowledgement | **Unassigned** |
| Secondary / Escalation Manager | Backup if primary is unavailable; may stop rollout | Name, contact and response expectation | **Unassigned** |
| Independent Approver | Reviews the evidence and authorizes Go; should not solely rely on the implementer | Name, approval timestamp and recorded decision | **Unassigned** |

## Required window record

Fill this out for the approved release; do not infer or fabricate people, paging routes, or acceptance.

- Date and start/end time with timezone: **TBD**
- Change / incident ticket: **TBD**
- Candidate commit SHA: **TBD until PR head is final**
- Candidate image digest: **TBD**
- Verified rollback image digest: **TBD**
- Monitoring dashboard and alert links: **TBD**
- Communication channel: **TBD**
- Primary and secondary on-call acknowledgement: **Pending**
- Independent go/no-go approver: **Pending**

## Escalation protocol

1. Anyone on the release team may call a **stop** when a gate fails or customer impact is suspected.
2. Primary On-Call acknowledges the page and alerts the SRE Lead and Release Owner.
3. Release Owner coordinates the decision; Application Owner diagnoses code behavior; Database Owner decides whether data/schema implications permit a binary rollback.
4. If the primary responder does not acknowledge within the team's documented response SLA, escalate to the Secondary On-Call, then the Engineering Manager / Incident Commander.
5. On a trigger condition, halt further rollout and execute the approved rollback plan. Do not continue rollout while health, dependency status, or telemetry is unknown.
6. Record timestamps, commands, actor, image digest, measurements, decision, and follow-up incident actions.

## Operational acceptance checklist

- [ ] Named primary and backup operators have explicitly acknowledged.
- [ ] Release Owner and independent approver are named.
- [ ] All required roles have working access to dashboards, CI, registry, cluster, and incident channel as appropriate.
- [ ] On-call coverage spans rollout plus the post-deploy observation window.
- [ ] Escalation SLA and contacts are documented and tested.
- [ ] Rollback performer and decision authority are clear.
- [ ] Customer-support / stakeholder communications are prepared.
- [ ] Change ticket references the pack and candidate digest.

**Approval gate:** This document intentionally does not invent names or declare role acceptance. Deployment remains blocked until assignments and acknowledgements are completed in the ticket/release record.