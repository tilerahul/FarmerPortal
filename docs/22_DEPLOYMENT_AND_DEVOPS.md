# Agri Procurement & Farmer Management System
## Deployment & DevOps Architecture

---

## 1. Document Information

| Item | Value |
|---|---|
| Product name | Agri Procurement & Farmer Management System |
| Document | Deployment & DevOps Architecture |
| Version | v1.0 |
| Status | Draft — architecture-level; infrastructure is **provider-neutral** (`[UNDEFINED]` until approved) |
| Date | 2026-09-16 |
| Author role | Senior DevOps / Cloud Architect |
| Purpose | Document the deployment architecture across Development → Staging → Production, per component (frontend, API, DB, storage, workers, integrations), plus CI/CD, observability, backup/DR/rollback, health, scaling and readiness checklists |
| Basis | `docs/18_SECURITY` (encryption, secrets, backups, monitoring), `docs/16_API`, `docs/15_DATABASE`, `docs/19_EXTERNAL_INTEGRATIONS`, `docs/21_TESTING`, scale facts `[SOURCE §1, §19]` |

### 2-Part separation (as requested)

| Class | Meaning | Examples in this doc |
|---|---|---|
| **REQUIREMENT** (Architecture requirement) | The deployment *must* do this regardless of host | TLS policy, encryption at rest, app/DB separation, secrets-not-in-repo, restore drills, auto-recovery health checks |
| **PROPOSED** (Infrastructure choice) | A candidate technology/host topology awaiting approval | cloud vs on-prem ASIDE, container runtime, scheduler names, monitoring stack |

> No hosting provider is assumed. Abstract terms are used; a mapping table (§24.0) lets a chosen provider be substituted without rewriting the architecture.

---

## 2. Environment Strategy

### 2.0 Environments (REQUIREMENT)

| Environment | Purpose | Data profile | Access |
|---|---|---|---|
| Development | Individual feature/dev builds | Synthetic, masked (OQ-18-consistent), small | Developers |
| Test/QA | Automated + manual testing (incl. `docs/21`) | Synthetic + scenario sets, masked | QA |
| Staging | Pre-prod validation: release candidate, integration sandbox, DR rehearsal, perf smoke | Masked / synthetic near-prod scale | QA + DevOps + business preview |
| Production | Live system (Phase 1: 10,000 farmers; Phase 2: 50,000 `[SOURCE §1, §19]`) | Production data | Admin/employee/farmer |

- (REQUIREMENT) Non-prod never uses production data unmas ked; sanitised pipeline per Q-SEC-11.
- (PROPOSED) symmetric topologies: staging mirrors prod component topology so deployment mechanics are identical.

### 2.1 Development Environment (REQUIREMENT + PROPOSED features)

| Aspect | Detail |
|---|---|
| Local dev topology (PROPOSED) | Developers run the three apps (portal frontend, API, DB) locally with containers; integration-facing features use provider sandboxes (WhatsApp/bank — `docs/19`) |
| Workspace (PROPOSED) | Feature branches → PR → CI; environment from branch for isolation |
| Data (REQUIREMENT) | Seeders / fixtures; no production data; masked copy cones where needed |
| Secrets (REQUIREMENT) | Local secrets via dev secrets manager with distinct values — no prod secrets in dev |

### 2.2 Staging Environment (REQUIREMENT + PROPOSED)

| Aspect | Detail |
|---|---|
| Topology (REQUIREMENT) | Full stack equivalent to prod (frontend, API nodes, DB, storage, workers) with provider sandboxes |
| Release path (REQUIREMENT) | Every release candidate passes staging: deploy → verification suite (`docs/21` §25 regression + smoke) → go/no-go |
| Drill venue (REQUIREMENT) | Performs backup/restore rehearsal (§18) and DR walkthrough (§19) |
| Parity (PROPOSED) | Config parity review tooling; drift detection between staging/prod config |

### 2.3 Production Environment (REQUIREMENT)

| Aspect | Detail |
|---|---|
| Multi-component, separated tiers | Portal/frontend, API nodes (scalable, stateless), dedicated DB, object/file storage, workers/queue — **app and DB never co-hosted** (REQUIREMENT; supports `docs/18` isolation) |
| Redundancy (PROPOSED attainment) | ≥2 nodes for stateless tiers, HA DB option, multi-AZ/zone where host supports (topology requirement: no single point of failure in component terms) |
| Environments/region | Single prod region Phase 1; DR capability per §19 (secondary copy; RPO/RTO pending Q-SEC-15/16) |
| Change window | FDS/mantainability-window tooling + approvals; audited deployments |

---

## 3. Component Architecture

```mermaid
flowchart LR
    W[Web/Portal: Admin, Employee, Farmer] --> LB[Load balancer]
    LB --> API[API nodes (stateless)]
    API --> DB[(Database)]
    API --> OBJ[(File / document storage)]
    API --> MSG[Queue]
    MSG --> WK[Background workers]
    WK --> WAPI[WhatsApp]
    WK --> BNK[Bank / API]
    OBJ --> WK
```

### 3.1 Frontend (Portal)
- **Type (REQUIREMENT):** responsive web SPA/PWA served over HTTPS for Admin, Employee, Farmer portals; mobile-browser based (`[NEW]`, `docs/17` §2) — no native app.
- **Deploy (PROPOSED):** static build artifact + CDN/cache; versioned builds; cache-busting; **no server-side session state in frontend**.
- **Security (REQUIREMENT):** served only over TLS, HSTS; no secrets in client bundle (`docs/18` §13, §16).

### 3.2 Backend / API
- **Type (REQUIREMENT):** versioned REST API under /api/v1 (`docs/16`); stateless nodes (session state kept out of process) → horizontal scaling (REQUIREMENT for §22).
- **Deploy (PROPOSED):** containers; orchestrator-compatible deployment (abstract) with rolling updates; health endpoints (§21).
- **Security (REQUIREMENT):** TLS, input validation at boundary, deny-by-default middleware, rate limiting (API), audit-safe logging.

### 3.3 Database
- **Storage engine (PROPOSED):** relational DB (transactional integrity; `docs/15` logical model); **technology-neutral SERVE — supplier TBD (Q-DB-01)**, no provider assumed.
- **Deployment (REQUIREMENT):**
  - App/DB separation; dedicated instance(s); separate credentials; encrypted at rest (`[SOURCE §18]`, `docs/18` §14 via KMS).
  - **Backups (§17) and point-in-time recovery configured; nightly verification.**
  - Data-tier scale: Phase 2 50,000 farmers / 500 employees (`[SOURCE §1, §19]`) + million-transaction order (`docs/21` SC-12); rollups/index strategy via `docs/15` DI-*, `docs/13` §16.

### 3.4 File / Document Storage
- **Purpose (REQUIREMENT if adopted):** PDF artefacts (statements/invoices), KYC docs (future), exports — private store, bracket-encrypted at rest, signed expiring URLs, AV-scanned uploads (`docs/18` §12/§19; `docs/19` INT-04).
- **Provision (PROPOSED):** object-store-class service or self-hosted equivalent; bucket/namespace segregation; lifecycle/retention rules per OQ-11.

### 3.5 Background Workers / Jobs
- **Jobs (REQUIREMENT):** 1st-of-month statement generation + dispatch (`[SOURCE §13]`), statement batch, notification send/retry (`docs/20`), reconciliation/bank ingestion (`INT-01`), report async generation (`docs/13`), KYC AV scan, digest/rollup computation.
- **Queue (PROPOSED):** durable queue; retry/backoff; dlq for failures (`[UNDEFINED]` worker/queue tech Q-DEP-05).
- **Operational notes (REQUIREMENT):** jobs are idempotent (no invoice renumber `/ [SOURCE §6]`; statement re-gen safe), tracked (job registry + audit), schedulable with pause on deployment windows.

### 3.6 WhatsApp Integration
- **Blueprint (REQUIREMENT):** portal ↔ WhatsApp provider via outbound messages + inbound status/webhooks (`[SOURCE §8, §12, §13]`; Phase 2 chatbot `[SOURCE §21]`; contract per `docs/19` INT-02).
- **DevOps aspects (REQUIREMENT):** provider credentials in secrets manager; webhook endpoints signed + replay-protected; sandbox credentials in non-prod; 1st-of-month burst capacity sized into worker pool (§22).
- (PROPOSED) three-way traffic isolation: dev/stage/prod credentials never shared.

### 3.7 Bank Integration
- **Blueprint (REQUIREMENT):** outbound payment init + inbound status/UTR + statement ingestion (`[SOURCE §10, §11]`; workflow `Portal → Bank/API → Payment → UTR/status → Portal`).
- **DevOps aspects (REQUIREMENT):** scoped API credentials/certs per environment; mTLS support; network allow-listing; idempotency namespace per env; signed webhooks; no bank secrets in app config.

---

## 4. Configuration Management

### 4.1 Environment Variables & 4.2 Secrets Management (REQUIREMENT)

| Concern | Handling |
|---|---|
| Env config | Non-secret config via env/config service per environment; env separation enforced (dev/stage/prod namespaces) |
| Secrets | Central secrets manager only (DB creds, bank, WhatsApp, TLS keys, signing keys); **never in repo, images, or client bundle** (`docs/18` §23); rotation + emergency rotation; audit of access |
| Which values are secrets (list — REQUIRED) | DB credentials, bank API key/cert, WhatsApp token, refresh-token signing key, TLS keys, KMS/key-refs, storage credentials, provider webhook secrets |
| 12-factor | Config via env; no env-dependent code branches beyond documented switches |

---

## 5. CI/CD (REQUIREMENT skeleton + PROPOSED tooling)

| Stage | Activity |
|---|---|
| Commit/Build | Branch build; lint/test gate; unit (T 1), integration (T 2); container scan (CVE), secret-scan |
| Test | API contract (T 3), UI E2E (T 4), security heavy path (T 21), perf smoke |
| Build artifact | Immutable versioned artifacts (build id = trace); image signing (trusted source) |
| Deploy staging | Auto on merge to release branch; run verification + regression (`docs/21` §25) |
| Gate/Approvals | Promotion to prod requires approval; change ticket + audit log |
| Deploy prod | **Rolling/canary** (REQUIREMENT: no full switchover; canary % then scale out); auto-rollback on health-gate failure (§20) |
| Traceability | Deploy event in audit log; artifact↔commit↔config drift tooling |

- (REQUIREMENT) No secrets/CI tokens in the pipeline definition; pipeline uses short-lived role tokens.
- (PROPOSED) pipeline tooling abstract; choose at Q-DEP-02.

---

## 6. Observability

### 6.1 Logging (REQUIREMENT)
- Structured, centralised logs; correlation by trace-id (`docs/18` §24); **PII/secrets never in clear** (mobile, UTR, bank account masked/excluded — `docs/18` §11/§20); retention per OQ-11.
- Sources: web/API access, app, workers/jobs, DB (audit), provider adapters — unified pipeline.
- (PROPOSED) log aggregation tooling abstract; forwarder config per env.

### 6.2 Monitoring & 6.3 Error Tracking (REQUIREMENT + PROPOSED coverage)
- Dashboards: auth failures, rate-limit hits, permission-denial surges, out-of-scope attempts, reconciliation queue depth, notification/statement job success, backup/encryption status, integration (bank/WhatsApp) latency & errors (`docs/18` §24).
- Alerts on breach thresholds (brute-force waves, webhook anomalies, backup failure, cert expiry, secrets rotation failure, queue backpressure).
- Error tracking: centralised exception aggregation with trace id and release version; error grouping dedups (no PII in stack payloads).

---

## 7. Backup, Restoration & Disaster Recovery

### 7.1 Database backups (REQUIREMENT — `docs/18` §21)
- Full + incremental + PITR window; **encrypted at rest** (backup copies); off-site/secondary copy of recent full; integrity hashes; retention aligned to recovery goals and OQ-11.
- Schedule/RPO: Q-SEC-15 (pending). RTO target: Q-SEC-16 (pending). These gates are REQUIRED to be defined before go-live (production readiness §24).

### 7.2 Backup restoration (REQUIREMENT — `docs/18` §22)
- Scheduled restore drills in staging (quarterly cadence); restore from real artifacts; integrity verification; attestation recorded; covered in `docs/21` §24.

### 7.3 Disaster recovery (REQUIREMENT skeleton + PROPOSED topology)
- DR plan documented: RPO/RTO targets (Q-SEC-15/16), recovery sequence (infra → API → data → storage), runbook, test cadence.
- (PROPOSED) secondary-region/zone passive or active standby; promoted on failover; storage cross-region copy for critical artifacts; DNS failover. Actual DR tier/prov + decisions pending Q-DEP-06/Q-SEC-16.

---

## 8. Rollback (REQUIREMENT)
- Every deploy is reversible:
  - **App:** artifact-pinned immutable builds; rolling rollback to prior artifact; health gates block new version promotion on failure; posture: old version stays until new is verified.
  - **Data:** apply migration strategy that is reversible (forward/backward migrations reviewed, additive-first where possible; destructive steps gated); schema/data migration runs before app deploy or with compatibility window — choice documented in release runbook (Q-DEP-01).
  - **Feature flag kill-switch** for high-risk toggles (REQUIREMENT pattern for permission-matrix/integration changes).
- Rollback is a **tested** runbook entry, rehearsed in staging (§19/§24).

---

## 9. Health Checks (REQUIREMENT)
- Liveness (process up) and readiness (dependencies reachable: DB, storage, queue, provider adapters) endpoints for API + workers.
- Load balancer uses readiness to drain/route; deploy gate uses health to auto-stop promotion; failed health → auto-remove instance + alert.
- Include background-job health: queue depth, stalled workers, DLQ count surfacing on dashboards.

---

## 10. Scaling (REQUIREMENT + PROPOSED)
- **Stateless API tier:** horizontal by node count; load-balanced; scale for 10,000/50,000 farmers + 500 employees (`[SOURCE §1, §19]`).
- **Background jobs:** worker pool sized for 1st-of-month statement batch burst + notification bursts (`docs/13` §16/§9, `docs/19`).
- **DB tier:** vertical headroom then read-replica/partition options per volume (Q-DB capacity decision); rollup/snapshot support.
- **Storage:** growth quota + lifecycle policy.
- **Autoscale (PROPOSED):** metric-based (queue depth, CPU) — provider-specific; deliberate so jobs remain idempotent under scale-out.
- Scale/load rehearsal every release at Q-DEP level: staging perf smoke against synthetic near-prod (per `docs/21` §22/23).

---

## 11. Deployment Checklist (per release)
Validate each item before promote:
1. Artifact pinned + signed; no secret in build/artifact (scan result attached).
2. Migrations reviewed; rollback path defined (Q-DEP-01).
3. Staging verification suite green (regression — `docs/21` §25; smoke incl. provider sandbox).
4. Health checks passing on all tiers; LB config parity.
5. Backup pre-deploy taken; tested restore on staging.
6. Config/env diff reviewed (drift tool clean); secret rotation risk assessed.
7. Alerting/monitoring dashboards updated for new version/build.
8. Rollback runbook current; feature flags point at new behaviour.
9. Security gates: container scan open, SAST/DAST findings ≤ threshold (Q-TST-03), rate-limit config reviewed.
10. Change record prepared (audited deployment record).

## 12. Production Readiness Checklist (release to live)
1. RPO/RTO documented (Q-SEC-15/16) and restore drill attested.
2. Encryption at rest verified on DB, storage, backups (`[SOURCE §18]`, `docs/18` §14).
3. TLS/HSTS verified; secrets manager access audited; no secrets in repo.
4. Provider integrations (bank/WhatsApp) in **prod-mode** credentials with sandbox sign-off; signature verification enabled.
5. Monitoring/alerts/live error-tracking active; retention configured (OQ-11).
6. Rollback + DR runbooks rehearsed in staging; go/no-go with owners.
7. Performance SLAs cleared (Q-TST-06/NFR); scale smoke at production target.
8. Backup schedule active with verification; off-site copy tested.
9. UAT sign-off (`docs/21` §26); audit trail empty-of-exceptions review.
10. Support/on-call handover documented; incident response owners named (Q-SEC-17).

---

## 13. Open Questions (DevOps/Deployment)

| ID | Question | Class |
|---|---|---|
| Q-DEP-01 | Migration strategy (additive-first; backward-compatible window) | REQUIREMENT-need |
| Q-DEP-02 | CI/CD tooling selection | PROPOSED |
| Q-DEP-03 | Hosting provider / on-prem posture + region(s) | PROPOSED |
| Q-DEP-04 | DR tier/site satisfaction (secondary zone vs off-site backups only) | PROPOSED |
| Q-DEP-05 | Queue/worker technology choice | PROPOSED |
| Q-DEP-06 | Storage service selection (object store vs self-hosted) | PROPOSED |
| Q-DEP-07 | Log/monitor/error-tracking stack selection | PROPOSED |
| Q-DEP-08 | Container/runtime & orchestrator choice | PROPOSED |
| Q-DEP-09 | Multi-tenancy/env separation model (namespace-per-env) | PROPOSED |
| Q-DEP-10 | Canary % / rolling policy numbers | PROPOSED |
| Reused | Q-SEC-15/16 (RPO/RTO/restore), Q-SEC-11 (sanitised backups), Q-SEC-14 (audit fail mode), Q-DB-01 (DB selection), Q-INT-03 (bank provider), Q-INT-09 (WhatsApp provider), OQ-11 (retention), Q-TST-06 (NFR SLAs) |

---

*End of Deployment & DevOps Architecture v1.0. Next in sequence: `23_NON_FUNCTIONAL_REQUIREMENTS_AND_SCALABILITY.md`.*