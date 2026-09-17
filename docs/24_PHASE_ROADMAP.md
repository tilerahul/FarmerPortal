# Agri Procurement & Farmer Management System
## Phase Roadmap

---

## 1. Document Information

| Item | Value |
|---|---|
| Product name | Agri Procurement & Farmer Management System |
| Document | Phase Roadmap |
| Version | v1.0 |
| Status | Draft — dependency-driven staging of Phase 1, Phase 2, and Future capabilities derived from the original PDF and all approved specs |
| Date | 2026-09-16 |
| Author role | Senior Solutions / Programme Architect |
| Basis | `[SOURCE §1–§24]` (esp. §19 scale, §20 future-ready, §21 chatbot, §22–§24 area); `docs/00` §7–§9; all specs `docs/01…23` |
| Rule | **No dates or deadlines are assigned.** Sequencing is dependency-based; releases are gated on completion criteria, not calendar. |

### Conventions

| Tag | Meaning |
|---|---|
| `[SOURCE §n]` | Statement from the original PDF |
| `[NEW]` | Newly approved Farmer Portal requirement (Phase 1) |
| `[PROPOSED]` | Proposed addition — requires explicit approval to enter any phase |
| `[UNDEFINED]` / OQ | Open question that gates a phase feature |

---

## 2. Roadmap at a Glance

| Phase | Content | Source basis | Nature |
|---|---|---|---|
| **Phase 1** | Core procurement, payment & statement system + Farmer Login / Farmer Web Portal | `[SOURCE §1]` core; portal `[NEW]` | Deliverable (core business) |
| **Phase 2** | WhatsApp Chatbot + Area-wise allocation, employee access, reporting | `[SOURCE §21–§24]` | Deliverable (scope extension) |
| **Future** | Open §20 capabilities + scale beyond §19 + other proposed items | `[SOURCE §19, §20]` + `[PROPOSED]` | Option queue (approval-gated) |

---

## 3. PHASE 1 — Core Procurement, Payment & Statement System + Farmer Login/Web Portal

### 3.1 Objective
Deliver the end-to-end transaction backbone (chain Farmer → Purchase → Invoice → Ledger → Payment → UTR → Monthly Statement → WhatsApp `[SOURCE §1]`) with a secure, own-data-only Farmer Web Portal (`[NEW]`), at Phase-1 scale (10,000 farmers / 100 employees `[SOURCE §1, §19]`).

### 3.2 Features
- Procurement & invoice workflow with invoice numbering uniqueness and cancellation rules (`[SOURCE §5–§7]`).
- Payment & bank reconciliation with statuses Matched/Unmatched/Failed/Pending/Duplicate and UTR capture (`[SOURCE §10–§11]`).
- Farmer ledger and monthly statement on the 1st with WhatsApp PDF delivery (`[SOURCE §9, §13]`).
- WhatsApp notifications for purchase, payment, statement (`[SOURCE §8, §12, §13]`) + in-portal notifications (`[NEW]`, `docs/20`).
- Farmer Login / Farmer Web Portal — own-data dashboard, profile, purchases, invoices, payments, ledger, statements, notifications, support (`[NEW]`; `docs/07`, `docs/17`).
- Admin & employee portals: dashboard KPIs, report centre (R-01…R-08) with PDF/Excel export, reconciliation queue, audit logs, settings/RBAC matrix (`[SOURCE §14–§16]`, `docs/13`, `docs/17`).
- Security, audit trail, backups, monitoring baselines (`[SOURCE §17–§18]`, `docs/18`).

### 3.3 Dependencies
- Decisions required before Phase 1 release (documented gates): farmer login method (OQ-02 / Q-SEC-02); payment allocation model (Q-PAY-006); outstanding/ledger formulas (Q-LS); bank & WhatsApp providers + status mechanism (Q-INT-03/09, Q-PAY-003); permission-matrix entries (OQ-09); retention (OQ-11); RPO/RTO (Q-SEC-15/16); DB technology (Q-DB-01).
- Feature-level: statement dispatch depends on WhatsApp integration + PDF artefact (INT-02/03); reconciliation queue depends on payment statuses; farmer portal depends on own-data-scoped APIs (P-ISO).

### 3.4 Business Value
- Core operational correctness (invoice numbering, ledger, statements) forms the trust base for farmers and staff.
- Farmer self-service (`[NEW]`) reduces call/office volume, gives farmers visibility of own balances/statements, increases transparency.
- Automation of statements/notifications reduces admin effort and late-information risk (`[SOURCE §13]`).

### 3.5 Technical Dependencies
- Stateless API with identity-scoped farmer endpoints (P-ISO-01…06; DBZ-01).
- Relational transactional core (`docs/15`); unique invoice index (`[SOURCE §6]`, TC-01).
- Queue/workers for notification + statement batch; provider sandboxes for bank/WhatsApp (`docs/19`, `docs/22`).
- Data tier sized for 10k→50k (`[SOURCE §19]`) headroom (`docs/23`).
- Secrets management, TLS, encryption at rest, audit append-only (`docs/18`).

### 3.6 Risks
| Risk | Mitigation |
|---|---|
| Open decisions (allocation model, formulas, providers) stall API/database locks | Gate these in the first decision cycle; specs already flag each (Q-*, OQ-*) |
| Reconciliation ambiguity (partial/multiple payment) delays PAY/LED modules | SC-04 scenarios scrubbed against decided model (Q-PAY-006) |
| Farmer portal scope creep beyond own-data read model | Keep Phase 1 farmer editing to OQ-18 decisions; no report centre for farmers |
| Statement/notification burst pressure at 10k farmers | Worker-pool isolation + pacing (Q-PRF-07) |
| Security gate findings late (auth/isolation) | Isolate first with the "own-data" axiom tested early (`docs/21` SC-05/06) |

### 3.7 Completion Criteria
- All FR-* for Phase 1 modules pass acceptance (`docs/21` §26).
- Financial-integrity scenarios SC-01…SC-06, SC-08…SC-11 green; no known P1/P2 (`docs/21` §6).
- Farmer portal E2E per role across breakpoints (`docs/17`).
- Backup/restore drill attested; RPO/RTO recorded (Q-SEC-15/16).
- Scale smoke at 10,000 farmers / 100 employees passes (Q-TST-06 budgets).
- Audit trail covers all §17 events; no secrets in build (`docs/18`).

### 3.8 Out of Scope (Phase 1)
- WhatsApp chatbot, area-wise allocation/access/reporting (→ Phase 2).
- All §20 future-ready capabilities (collection centre, warehouse, export shipment, quality/grade/GPS etc.) — surfaced only where pre-positioned (`docs/15` E-13…E-18).
- Report R-09 area-wise; employee area-scoping; SMS/email/push channels (unless approved as fallback).
- Report scheduling/drill-down, invoice PDF delivery unless OQ-07 decided.

---

## 4. PHASE 2 — WhatsApp Chatbot, Area-wise Allocation, Area-wise Employee Access, Area-wise Reporting

### 4.1 Objective
Extend the portal into a digital interaction channel via WhatsApp chatbot (`[SOURCE §21]`) and add the farm-area hierarchy so employees access and report strictly within their assigned areas (`[SOURCE §22–§24]`), at scale to 50,000 farmers / 500 employees (`[SOURCE §19]`).

### 4.2 Features
- **WhatsApp Chatbot** (`[SOURCE §21]`): 9 command set (`docs/11`), own-data responses (P-ISO), menu/onboarding, channel integration extension of Phase 1 send pipeline.
- **Area-wise Farmer Allocation** (`[SOURCE §22]`): area entity, farmer-to-area assignment, transfer/reassignment with audit.
- **Area-wise Employee Access** (`[SOURCE §22–§23]`): employee-area assignment, DB-level scoping (E-AR-01…04, DBZ-02/03) for all employee queries incl. exports.
- **Area-wise Reporting** (`[SOURCE §24]`): R-09 farm area-wise procurement; area-metrics dashboard extension (reports P2).

### 4.3 Dependencies
- Requires Phase 1 stable transaction+audit core (area scoping rides on the same scoped-query framework, FRPT-01/02).
- Chatbot requires: WhatsApp outbound integration operational; command semantics (Q-WH-05…07); own-data guard testing (SC-07).
- Areas require: area master + assignment UI; data migration/upsert path for farmer-area mapping; reporting rollups keyed by area; access-matrix entries for area scope (OQ-09).

### 4.4 Business Value
- Farmer access/self-service via the channel they already use on phones ($13 habits) — expected to raise engagement and reduce staff queries (business-value statement; figures `[PROPOSED]`).
- Area governance improves ownership/accountability, localised service quality, and management visibility per area.
- Area-wise reporting gives operational control granularity for expansion to 50,000 farmers.

### 4.5 Technical Dependencies
- Area attribute on FARMER + EMPLOYEE + scope predicate push-down (DBZ-02/03) — no filter-after-fetch (`docs/12`, `docs/15` E-17/E-18 Phase 2).
- Master-area assignment API + audit events; area scoping inside report/export pipeline (FRPT-02).
- Chatbot session/state handler with own-data scoping; provider webhook signing; rate-lane isolation (`docs/19` INT-02; `docs/23` §14).
- Scale readiness of statement/notification pipeline at 50,000 farmers (`docs/23` §18–19).

### 4.6 Risks
| Risk | Mitigation |
|---|---|
| Chatbot own-data leakage | P-ISO guard in all chat responses; SC-07-style scenarios for chat; no cross-farmer state |
| Area rule bypass in raw queries | Scope push-down at DB layer; positive/negative isolation tests (`docs/21` §7) |
| Migration cost of assigning 10k–50k farmers to areas | Design import/bulk-assignment tooling and assistive assignment (proposed in Phase 2 entry) |
| Concurrent employee scale (500) writing during entry peaks | Stateless horizontal API + worker lanes (`docs/23` §4) |
| Chat semantics pre-date definitions | Deliverables blocked until Q-WH-05…07 decided |

### 4.7 Completion Criteria
- Chatbot command set working with own-data isolation attested (`docs/11` runbook).
- Area allocation/access/transfer flows auditable and enforced at DB level (SC-07 green; negative cases).
- R-09 + area dashboards match area-scoped source data; exports carry area scope (FRPT-02).
- Scale test at 50,000 farmers / 500 employees passes budgets (Q-PRF-02).
- No regression in Phase 1 financial/statement paths.

### 4.8 Out of Scope (Phase 2)
- §20 future-ready capabilities; SMS/email channels unless approved; report scheduling (Q-RPT-02); chatbot beyond commanded scope (e.g., free natural-language purchase entry); area invention/creation governance beyond the master.

---

## 5. FUTURE — Source-Explicit + Proposed Options

> Only capabilities explicitly mentioned in the original PDF, or clearly marked `[PROPOSED]`, are listed. No dates assigned; each is **option-gated** by approval.

### 5.1 Source-explicit future capabilities
| Item | Source | Approval gate |
|---|---|---|
| Collection centre / warehouse / export shipment modules | `[SOURCE §20]` future-ready fields | Data-model pre-positioning exists (`docs/15` E-13…E-18); needs product decision |
| Quality / grade / GPS / lot / batch attributes on procurement (provenance, grading) | `[SOURCE §20]` | Schema evolves backward-compatibly; affects product/master reports |
| Scale beyond 50,000 farmers / 500 employees (higher volume headroom) | `[SOURCE §19]` FDS line | Driven by capacity plan (`docs/23` §18); architecture readiness `[PROPOSED]` |
| KYC-supported farmer onboarding (documents in master) | `[SOURCE §20]` future-ready | Depends on document storage adoption (Q-INT-16) and `docs/18` file-upload security |
| Higher SMS/email/other channels as supplementary notifications | `[NEW]`-adjacent / market-driven `[PROPOSED]` | Only if approved; retained strictly as `[PROPOSED]` in `docs/20` |
| In-portal farmer self-service enhancements (KYC update, address edit, receipts) | `[NEW]`/`[PROPOSED]` (OQ-18) | Depends on OQ-18 edit-scope decision |

### 5.2 Proposed future capabilities (must be approved; not source-required)
| Item | Proposal ref | Gate |
|---|---|---|
| Report scheduling, saved filters, drill-down, distribution | `docs/13` Q-RPT-02/04 | Q-RPT decisions |
| CSV export / quantity & rate-wise report variants | Q-RPT-07/08 | Approval |
| Digital signatures on statements/invoices | Q-RPT-02 (digitised signatures) | Approval + legal review |
| SMS/email fallback channels | `docs/20` Q-NTF-08/09; `docs/19` | Approval |
| OTP-based farmer login enhancement | `docs/18` §5; Q-SEC-02 | Auth decision |
| Object-store KYC + scanning pipeline | `docs/18` §12/§19; INT-04 | Q-INT-16/17 |
| Archival/purge automation for aged data | `docs/23` Q-PRF-10; OQ-11 | Retention decision |

### 5.3 Objective (Future)
Keep the system **expansion-ready**: pre-positioned data model, scoped-query foundation, and reversible architecture that can adopt approved §20 capabilities and higher-volume scale without redesign.

### 5.4 Dependencies, Risks, Completion Criteria (Future)
- **Dependencies:** each item requires its approval gate + prior-phase foundations (audit, scope framework, statement/notification pipeline, storage).
- **Risks:** adopting §20 items without product decision dilutes focus; schema evolution risk mitigated by additive/backward-compatible changes (`docs/22` Q-DEP-01).
- **Completion criteria (per item):** feasibility validated against Phase-1/2 foundations; approved spec exists; acceptance through standard pipeline (`docs/21`).
- **Out of scope:** anything not in §5.1/§5.2 remains excluded unless a business-approved requirement is raised.

---

## 6. Cross-Phase Gates, Decision Log Alignment

| Gate type | Examples | Where owned |
|---|---|---|
| Business decisions | Q-PAY-006, OQ-02, OQ-08, OQ-09, Q-INT-03/09 | Open Questions & Decisions log |
| Financial definitions | Q-LS outstanding/statement formulas | `docs/10` |
| Security/ops | Q-SEC-15/16 RPO-RTO, Q-SEC-14, Q-DEP-01 | `docs/18`, `docs/22` |
| Scale objective | Q-PRF-02, Q-TST-06 | `docs/23`, `docs/21` |
| Data/tech | Q-DB-01…03 | `docs/15`, `docs/22` |

---

*End of Phase Roadmap v1.0. Next in sequence: `25_GLOSSARY_AND_TERMINOLOGY.md` (optional) or index reconciliation.*