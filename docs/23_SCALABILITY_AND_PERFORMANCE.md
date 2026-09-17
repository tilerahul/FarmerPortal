# Agri Procurement & Farmer Management System
## Scalability & Performance Specification

---

## 1. Document Information

| Item | Value |
|---|---|
| Product name | Agri Procurement & Farmer Management System |
| Document | Scalability & Performance Specification |
| Version | v1.0 |
| Status | Draft — scale and performance architecture grounded in the original requirements |
| Date | 2026-09-16 |
| Author role | Senior Performance / Scalability Architect |
| Purpose | Document scalability objectives and performance architecture covering user, transaction, database, reporting, background, integration and monitoring dimensions |
| Primary source | `AgriProcurement & Farmer Management.pdf` — §1, §19, §22, §24 (scale and area growth), §13 (statement) |
| Aligned specs | `docs/13` (§16 performance), `docs/15` (data model DI-* index guidance), `docs/16_API`, `docs/21` (§22–23, SC-12), `docs/22` (deployment/scaling) |

### Tiers & source basis

| Fact | Value | Source |
|---|---|---|
| Initial (Phase 1) farmers | ~10,000 | `[SOURCE §1, §19]` |
| Initial employees | 100 | `[SOURCE §1, §19]` |
| Scale target — farmers | 50,000 | `[SOURCE §1, §19]` |
| Scale target — employees | 500 | `[SOURCE §1, §19]` |
| Transaction density | "potentially millions of transaction records" — user-stated design target (derived from `docs/21` SC-12); not an explicit §19 figure | user requirement |
| Monthly statement cadence | 1st of every month | `[SOURCE §13]` |

### Conventions

| Tag | Meaning |
|---|---|
| `[SOURCE §n]` | Original PDF statement |
| `[PROPOSED]` | Proposed assumption/model — needs approval; **does not claim exact figures/SLAs** |
| `[UNDEFINED]` / OQ | Open decision (explicit targets, thresholds, provider) |
| Absent labels | Architecture guidance, not numbers |

> **Design-for-transaction-volume principle:** capacity is planned on transaction throughput and data growth (procurement, payments, ledger entries, statements, notifications), never only on farmer headcount. Farmer count is a driver of transaction volume, not a proxy for it.

---

## 2. Scalability Objectives

1. Launch Phase 1 at ~10,000 farmers / 100 employees (`[SOURCE §1, §19]`) with idle headroom for growth spikes (seasonality) `[PROPOSED]`.
2. Scale predictably, without redesign, to 50,000 farmers / 500 employees (`[SOURCE §1, §19]`) and the **millions-of-transactions** order (per §4).
3. Absorb **transaction-volume** growth independently of farmer-count growth: same 10,000 farmers with doubled procurement/payment rate must not degrade (Q in §8/§20).
4. Meet 1st-of-month statement burst (`[SOURCE §13]`) without degrading transaction availability.
5. Keep query/aggregation work on server pre-computation (rollups/snapshots `docs/13` §16), never client-side derivation.
6. Maintain audit immutability under high write volume (`docs/14`; million-row audit tables tested).

---

## 3–5. Growth Dimensions

### 3. Farmer Growth (10,000 → 50,000)
- **Driver:** user-visible scope growth (`[SOURCE §1, §19]`); Phase 2 adds area hierarchy (`[SOURCE §22]`) → farmer data is area-scoped.
- **Architecture effect:** storage and lookup scale by farmer count; dashboard counts (K-01…K-03) must stay O(1)-ish via **materialised counts/rollups**, not full scans (`docs/13` §16, `docs/15` DI-*).
- **Spring-point:** farmer master is primary-driver table; its write rate is low (registration), its read rate feeds all lists/dashboards/reports — read-optimised indexing required (§8).

### 4. Employee Growth (100 → 500)
- **Driver:** `[SOURCE §1, §19]`; 500 concurrent-ish entry users (Q: concurrent profile `[PROPOSED]`, Q-PRF-05).
- **Architecture effect:** entry concurrency drives **write-path** load (procurement creation at peak); API tier must be horizontally scaled with stateless sessions (`docs/22` §3.2) — 500 employees create *much more* write load than 500 farmers reading.
- **PK:** employees each open during working hours; purchase entry is the hot create path → transaction-throughput design (§6).

### 5. Transaction Growth (Millions)
- **Driver:** each procurement → invoice → ledger entry; each payment → ledger + possibly allocations; monthly statement per farmer; notifications; audit rows per mutation (`docs/14`).
- **Volume model `[PROPOSED]` (examples, not SLAs):** a month with 250,000+ purchase/invoice rows, similar payment/allocation scale, ~50,000 statements, plus multiple audit rows per event → mid-100k to millions of rows/month; cumulative ledger+audit tables reach the **millions** order quickly.
- **Architecture response:** 
  - Write path: batched/async for non-real-time (statements, notifications, report aggregates).
  - Read path: pre-computed rollups, snapshots, partitions; bounded pagination; capped exports (`docs/13` §16).
  - Audit (append-only, high-volume) separated/partitioned so it never blocks transaction reads (`docs/14`; Q-PRF-06).
- **Derived truth:** report/balance reads never ship full-history scans to the API layer.

---

## 6. Database Scalability

| Dimension | Approach | Class |
|---|---|---|
| Relational core | Transactional DB with ACID; logical model `docs/15` (E-01…E-18 + Phase 2) | `[UNDEFINED]` tech choice (Q-DB-01); REQUIRED relational semantics |
| Vertical headroom | Initial sizing sustains Phase 1 with headroom | `[PROPOSED]` |
| Horizontal path | Read-replica / partitioning options sequenced by volume triggers (Q-DB capacity decision `docs/22`) — NOT pre-committed infrastructure | `[PROPOSED]` |
| High-volume tables | Ledger entries, audit log, notifications: partitioning + rollup/snapshot tables reduce hot-scans | `[PROPOSED]` architecture (vendor-neutral) |
| Concurrency | Write serialisation where totals matter (allocation, invoicing) with optimistic/transactional control (`docs/15` TC-*) | `[PROPOSED]` |

---

## 7. Query Performance (REQUIREMENT posture)
- Every read path stated by reported queries (farmer lookup, ledger, statement, dashboard) pays for index access (§8) or a precomputed aggregate — never a full-table scan at volume.
- Query patterns derive from API/resource contract (`docs/16`): farmer-scoped `/farmer/me/*` reads are high-frequency, must stay single-row/tenant-aware (`docs/15` DBZ-scoped predicates).
- Pagination mandatory for lists; exports stream and cap (`docs/13` §16).
- No client-derived money math (data-layer canonical) (`docs/13` §14).

---

## 8. Indexing (with the million-row concern)

Base index set (extends `docs/15` DI-*):

| Table (logical) | Suggested indexes | Purpose |
|---|---|---|
| FARMER | PK, (status), (area_id) Phase 2, (mobile) unique, (name) | list/dashboard, area scope |
| EMPLOYEE | PK, (role), (status), (area_id) Phase 2 | lookup + scope |
| PROCUREMENT | (farmer_id, date), (date), (product_id, date), (employee_id, date) | heated report/ledger paths |
| INVOICE | (farmer_id, seq) **unique — numbering guard `[SOURCE §6]`**, (date), (status) | numbering, register |
| PAYMENT | (farmer_id, date), (utr) index, (status, date), (payment_id) | register, reconcile, PM views |
| PAYMENT_ALLOCATION | (invoice_id), (payment_id) | allocation queries |
| LEDGER_ENTRY | (farmer_id, entry_date), (period) | ledger + statement snapshot source |
| MONTHLY_STATEMENT | (farmer_id, period) unique | statement retrieval |
| AUDIT_LOG | (actor, date), (entity, record_ref, date), (action, date) | audit browser, immutability probes |
| WHATSAPP_MESSAGE / NOTIFICATION | (farmer_id, date), (status, created_at), (message_id) | notification tracking, retry scans |

- Index carefully on **write-heavy tables** (PROCUREMENT, LEDGER_ENTRY, AUDIT_LOG): coverage vs write amplification trade-off reviewed at Q-DB-02/03; unnecessary indexes removed to protect write throughput (SC-12 reality).
- **Millions-rows audit caveat:** audit browser queries (actor/date) need support indexes; append-only writes must not contend with the read oracle.

---

## 9. Reporting Performance
- Report reads hit **rollup/snapshot tables** (daily/monthly aggregates) rather than live tens-of-millions row scans (`docs/13` §16; P-01/P-02/P-03 feeds).
- Statement report (F-05) reads the pre-generated MONTHLY_STATEMENT snapshot (`docs/13` §16 F-05), not the live ledger — identical to the statement job output.
- Outstanding (F-03/K-09) uses as-of ledger snapshots; recompute scheduled, never at request time when volume heavy.
- Exports (PDF/Excel/CSV) async for large ranges; generation server-side; results capped and streamed (`[PROPOSED]`; `docs/13` §9, §16).
- Dashboard KPIs (K-01…K-11) served from materialised counts with periodic refresh (Q-RPT-05), isolated from OLTP writes.

---

## 10. Large Ledger Performance
- One farmer with many entries over years → ledger reads **period-windowed** (page by page, filter date) — never one unbounded row dump.
- Ledger **running-position** query must not re-sum the full history per request `[PROPOSED]`: monthly snapshot points (per `MONTHLY_STATEMENT`/rollup) provide a baseline; position = baseline + incremental window (aligned to `docs/10` derived-position design; formula remains Undefined — performance design is formula-independent).
- Farmer portal My Ledger (`[NEW]`) is a **slice** (own farmer, page/period) — must stay fast even while the farmer has tens of thousands of entries cumulative (SC-12 target).

---

## 11. Payment Reconciliation Performance
- Bank statement ingestion (INT-01): parse → match → status updates for **thousands of rows** per batch — process in streaming batches in background jobs; never per-transaction synchronous round-trips from UI (`docs/19` INT-01 §3.8).
- Matching operates on indexed PAYMENT (UTR) + stored statement data; set-based matching (multi-row) preferred over per-row loops `[PROPOSED]`.
- Reconciliation queue statuses (`[SOURCE §11]`) drive retry work; rate-limits respected (§14).
- 1st-of-month statement remains decoupled from reconciliation volume; both run as separate job pools (`docs/22` §3.5).

---

## 12. Monthly Statement Generation (burst)
- Source: statement on 1st of the month (`[SOURCE §13]`) to each farmer — the largest **controlled burst** in the system.
- Design:
  - Generation batch (worker pool) creates MONTHLY_STATEMENT snapshots from ledger rollup baselines (per §10) — O(N) where N=farmer count, from precomputed inputs.
  - Dispatch batch (WhatsApp/portal) shares provider rate limits (`docs/19` INT-02 §4.10; `[PROPOSED]` spread/pacing).
  - Generation and dispatch are separate job units; generation failure re-runs idempotently (no renumbering `[SOURCE §6]` guard), dispatch retries per `docs/20`).
  - Statement jobs run in a dedicated worker pool so transaction entry is unaffected during the 1st-of-month spike.
- Throughput/percentile/pacing numbers `[PROPOSED]` only (Q-PRF-07); do not claim SLAs.

---

## 13. Background Processing
- Queue (vendor-neutral; choice Q-DEP-05) with durable jobs, retry/backoff, dead-letter review (`docs/22` §3.5).
- Job classes: statement batch, notifications, reconciliation/ingestion, report gen, rollup recompute, AV scans, exports.
- Jobs idempotent; concurrency controlled so aggregate jobs don't contend with OLTP on DB.
- Priority lanes: entry-path safety events (payment status finalisation) above low-urgency (rollups).

---

## 14. WhatsApp Processing
- Outbound events have **permission-intensity**: 1st-of-month statement batch (huge), purchase/payment notifications (transactional volume) + Phase 2 chat (interactive).
- Rate limiting at portal side — respect provider per-day/session ceilings (`docs/19` INT-02 §4.10; expiry/undelivered handling per `docs/13` §13-conditions) — with pacing scheduler `[PROPOSED]`.
- Webhook status ingestion (Sent/Delivered/Failed/Retry) is async, idempotent, low-priority lane; never blocks outbound dispatch.
- Inbound Phase 2 routes to per-farmer scoped handlers (P-ISO guard) — concurrency per farmer modest while N farmers spikes totals (Q-PRF-09).

---

## 15. Caching Considerations `[PROPOSED]`
- **Read cache** for: dashboard KPIs (refresh Q-RPT-05), farmer list page-1/filters (short TTL), product catalogue, configuration settings, reports rollups.
- **Never cache:** financial balances as a source of truth; ledger correctness; audit; anything scope-dependent cross-tenant (cache keys MUST include tenant/identity scope — worker-data isolation).
- **Write-through/invalidation:** cache update on mutation, not TTL-only, for money-bearing card counts; fallback to recompute on miss.
- Cache infra choice `[UNDEFINED]` (Q-PRF-01); provider-neutral.

---

## 16. API Performance
- `/farmer/me/*` reads are the highest-rate farmer endpoints: single-farmer scoped, indexed, small payloads (`docs/16` P-ISO pattern).
- Entry/write endpoints (procurement create, payment confirm) must bound synchronous work: DB commit + audit + enqueue async side-effects (notify/statement) (queued — `docs/20`).
- Pagination, compression, ETags, request size caps, rate limits standardised (`docs/16`/`docs/18` §17-§18).
- Response time budgets `[PROPOSED]` (Q-PRF-02); not claimed as SLA.

---

## 17. Monitoring (Performance-specific)
- Metrics: request latency/throughput per endpoint tier, DB query times + slow queries, queue depths, job durations, statement batch progress, notification/WhatsApp delivery success/retry rates, reconciliation lag/backlog, cache hit rate, index bloat (Q-PRF-03).
- **Volume-adjusted alerts** — track per-transaction-rate thresholds, not static farmer-count proxies.
- Correlate via trace id across API/worker/provider (`docs/18` §24).
- Capacity trend dashboards feed §18 (auto-rollback/staleness per `docs/22` §6).

---

## 18. Capacity Planning
- Model on **transaction rates** (procurement/hr at peak season, payments/hr, notification commits) per 10,000/50,000 farmers and 100/500 employees — not farmer count alone.
- Inputs: seed & growth, seasonal peak multiples `[PROPOSED]`, statement burst, reconciliation batch volume.
- Outputs: DB sizing (rows/data per period → partitions/retention), API node count, worker pool size (statement/notification), storage growth for PDFs/KYC, export volume.
- Produced at release gates (`docs/22` §11/12) and re-run when source data changes shapes (Q-PRF-04).

---

## 19. Performance Testing (ties `docs/21` §22–23)
- Test tiers: API latency/throughput at Phase-1 and Phase-2 volume profiles; background-job throughput (statement batch at 50,000, notification burst); DB at million-row scans (index effectiveness SC-12); concurrency at 500 employees (`[SOURCE §1, §19]`).
- Workload profiles built from transaction models (§18), not synthetic-only shots; soak + spike + burst + recovery.
- Performance baselines ratify against thresholds set in Q-PRF-02 (no claimed SLAs yet).
- Perf smoke in staging on every release (`docs/22` §11/12).

---

## 20. Bottleneck Identification
- Systematic drill when thresholds are breached: DB lock/slow-query, write contention on allocation/invoice sequences, queue backpressure (statement/notification), provider throttle (WhatsApp/bank), cache thrash, logging volume (audit write amplification).
- Each identified bottleneck maps to a designed remedy (index/partition/rollup/queue re-pacing/scale-out add capacity) per `docs/22` scaling.
- Post-mitigation re-benchmark; findings captured to capacity plan (§18).
- Known hotspots to watch with millions of rows: AUDIT_LOG writes, LEDGER_ENTRY scans, INVOICE sequence contention, unpartitioned notification tables.

---

## 21. Future Scalability (beyond documented Phase-2 scale)
- Beyond 50,000 farmers / 500 employees (`[SOURCE §19]` future-ready direction): consider data partitioning by area (`[SOURCE §22]` hierarchy as partition key), read-scaling via replicas for reporting, archival/partition for old audit, eventual provider-based horizontal pipeline for statement dispatch, and telemetry-driven autoscaling.
- Kept as **architecture readiness**, configurable and reversible — not pre-committed `[PROPOSED]` until required (aligns `docs/22` §22; plug-in non-breaking).
- Independence-of-concern: farmer-count limits never bound transaction-volume headroom (the §2 objective #3).

---

## A. Open Questions

| ID | Question | Class |
|---|---|---|
| Q-PRF-01 | Cache layer technology choice | `[PROPOSED]` |
| Q-PRF-02 | Response-time/throughput budgets per tier | `[PROPOSED]` — user must approve figures |
| Q-PRF-03 | Metric retention/volume logging scope | `[PROPOSED]` |
| Q-PRF-04 | Capacity review cadence trigger | `[PROPOSED]` |
| Q-PRF-05 | Concurrent employee profile at peak (500 active?) | `[UNDEFINED]` |
| Q-PRF-06 | Audit-log read isolation approach (partition) | `[PROPOSED]` |
| Q-PRF-07 | Statement burst throughput/pacing targets | `[PROPOSED]` |
| Q-PRF-08 | Seasonal peak multiples applied in capacity model | `[PROPOSED]` |
| Q-PRF-09 | Phase 2 WhatsApp chat concurrency/lanes | `[PROPOSED]` |
| Q-PRF-10 | Archive policy for aged ledger/audit (beyond retention OQ-11) | `[UNDEFINED]` |
| Reused | Q-DB-01/02/03 (DB choice, index/partition), Q-RPT-05 (dashboard refresh), Q-DEP-05 (queue), Q-TST-06 (NFR SLAs), Q-SEC-15/16 (RPO/RTO), OQ-11 (retention) | |

---

## B. Source Reaffirmation

| Fact | Source |
|---|---|
| Initial 10,000 farmers / 100 employees | `[SOURCE §1, §19]` |
| Scale target 50,000 farmers / 500 employees | `[SOURCE §1, §19]` |
| System "should be designed in such a way that it can handle higher volume in the future" | `[SOURCE §19]` |
| Monthly statement on 1st of each month | `[SOURCE §13]` |
| Phase 2 farm-area growth | `[SOURCE §22, §24]` |

---

*End of Scalability & Performance Specification v1.0. Next in sequence: `24_NON_FUNCTIONAL_REQUIREMENTS_AND_SCALABILITY.md` decided by index.*