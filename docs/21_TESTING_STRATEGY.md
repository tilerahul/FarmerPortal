# Agri Procurement & Farmer Management System
## Testing Strategy

---

## 1. Document Information

| Item | Value |
|---|---|
| Product name | Agri Procurement & Farmer Management System |
| Document | Testing Strategy |
| Version | v1.0 |
| Status | Draft — strategy, levels, and critical business-rule test scenarios derived from the documented requirements |
| Date | 2026-09-16 |
| Author role | Senior QA Architect |
| Purpose | Complete testing strategy across unit, integration, API, UI, security, performance, backup/restore, regression and UAT, with detailed scenarios for the critical business rules |
| Basis | `AgriProcurement & Farmer Management.pdf` §1–§24 and spec set: `docs/01…20`, esp. `docs/06` (FR-*), `docs/09` (payment/reconciliation), `docs/10` (ledger/statement), `docs/11` (WhatsApp), `docs/12` (RBAC), `docs/13` (reports), `docs/14` (audit), `docs/15` (data model), `docs/16` (API), `docs/18` (security), `docs/20` (notifications) |
| Constraints | No test code is authored in this document; scenario definitions only. No specific test-tooling vendor mandated. |

### Conventions

| Tag | Meaning |
|---|---|
| `[SOURCE §n]` | Statement from the original PDF |
| `[NEW]` | Newly approved Farmer Portal requirement |
| `[PROPOSED]` | Assumption the QA plan depends on — requires approval |
| `[UNDEFINED]` / OQ | Open item; QA gates on the decision |
| FR-*, OQ-* | Requirement identifiers from `docs/06`, open questions log |

### Traceability note
Every scenario references requirement IDs. Where behaviour is undefined (marked in prior specs), the test asserts only the **currently documented** behaviour and flags the gap (Q-TST-*), never invents a rule.

---

## 2. Testing Objectives

1. Verify every documented functional requirement (FR-*) and source-stated behaviour (`[SOURCE §n]`) works end-to-end.
2. Prove **financial integrity**: quantity × rate, invoice numbering uniqueness, cancellation non-reuse, payment allocation, ledger correctness, statement correctness.
3. Prove **security posture**: authentication, RBAC, deny-by-default, and the two critical data-isolation guarantees — farmer own-data only (P-ISO-01…06) and Phase 2 employee area restriction at DB level (DBZ-02/03, E-AR-01…04).
4. Prove **auditability**: every required event is captured, immutable, complete (`[SOURCE §17]`).
5. Prove **operational resilience**: failure, retry, timeout, rate-limit, backup/restore and recovery behaviour.
6. Prove **scale-readiness**: system meets documented scale (10,000→50,000 farmers; 100→500 employees `[SOURCE §1, §19]`) and the million-transaction order of magnitude (Q-TST-20).
7. Provide **release confidence** for Phase 1 with a defined UAT exit path.

---

## 3. Test Strategy Overview (Pyramid)

| Level | Focus | Owner | Primarily verifies |
|---|---|---|---|
| Unit | Business rules in isolation (calc, numbering, status transitions) | Dev | FR-PRO/INV/PAY/LED, `[SOURCE §5, §6, §9]` |
| Integration | Module boundaries, DB constraints, async pipelines (WhatsApp, reconciliation) | Dev + QA | DBZ/TC constraints, `docs/14` |
| API | Contract, status codes, auth, scope | QA | `docs/16` |
| UI (Admin/Employee/Farmer) | Flows, states, responsive | QA | `docs/17` |
| Security | Auth, RBAC, isolation, web/API attacks | Security QA | `docs/18`, `docs/12` |
| Performance/Scalability | Load, volume, concurrency | Perf QA | `[SOURCE §19]` |
| Regression + UAT | Full suite + business sign-off | QA + BA | acceptance |

Entry criteria per level: prior level green, environments provisioned, test data seeded. Exit: defect severity ≤ threshold (Q-TST-03).

---

## 4. Test Environment & Data Strategy

- **Environments:** dev → QA/Test → staging (prod-like) → UAT → prod. Integration-sandbox for bank/WhatsApp providers (`docs/19`).
- **Test data:** seeded master sets (farmers, employees, products, areas Phase 2), plus **scripted ledger-historion** spanning multiple months to support statement and outstanding testing.
- **Isolation data:** include cross-farmer and cross-area records deliberately, to assert non-leakage (SC-05…SC-07).
- **Production-data sampling for perf:** 50,000 farmers / 500 employees per `[SOURCE §1, §19]`; synthetic transaction generation to the million-transaction order (Q-TST-20).
- **Masked/anon test data** matching OQ-18 display rules so UI assertions match production masking.

---

## 5. Test Sections

### 1. Unit Testing
- **Objective:** verify individual business rules deterministically — calculation, number formatting, status transitions, permission helpers.
- **Scope:** quantity × rate = gross → deduction → net (`[SOURCE §5 OR §9]` per `docs/09`); invoice numbering builder; UTR/status mapping; ledger posting helper; statement serializer; API validators.
- **Approach:** table-driven cases incl. boundary values, zero, negatives (rejected), decimals/rounding, currency thresholds. Coverage gate per module (Q-TST-01).
- **Exit:** all unit suites green; coverage gate met; no code-level review open.

### 2. Integration Testing
- **Objective:** verify modules work together and DB constraints hold (FKs, unique invoice index, audit immutability).
- **Scope:** Procurement→Invoice→Ledger; Payment→Allocation→Ledger→Statement; Reconciliation→status transition→WhatsApp trigger; webhook/status ingestion (`docs/19` INT-01/02); object/PDF storage handoff (INT-03/04).
- **Key constraints under test:** unique (farmer_id, sequence) invoice index (SC-01/02); ledger immutability (SC-10); scoped predicates DBZ-01…03 (SC-05…07).
- **Exit:** integration suite green incl. DB constraint negative tests.

### 3. API Testing
- **Objective:** verify `docs/16` contracts.
- **Scope:** all 17 groups / ~47 endpoints: method, status codes (401/403/404/409/422), query/path/body validation, idempotency keys (OQ-13), pagination, error model with trace-id, response shapes, webhook signatures (placeholder per Q-PAY-003).
- **Approach:** contract tests per endpoint; negative/validation matrix; auth-scope matrix (each role × each endpoint).
- **Exit:** contract suite + scope matrix green; no unauthorised or out-of-scope 200s.

### 4. UI Testing
- **Objective:** verify Admin, Employee, Farmer portals against `docs/17`.
- **Scope:** navigation/IA, all screens' states (loading/empty/error/success), guided purchase flow stepper (E3), farmer mobile-first layout, tablet/desktop breakpoints, dashboard KPIs render.
- **Approach:** component tests + end-to-end flows per portal; responsive screenshots at breakpoints; accessibility sanity (WCAG AA) `[PROPOSED]`.
- **Exit:** E2E suites green at all breakpoints; empty/error states verified on every screen.

### 5. Authentication Testing
- **Objective:** verify login, sessions, credential rules.
- **Scope:** employee/admin login (`[SOURCE §2]`), strong password/no default (`[SOURCE §4, §18]`), farmer login (`[NEW]`, OQ-02 — OTP `[PROPOSED]` or credential), lockout/throttle (FR-AUTH-006), 2FA for privileged (FR-AUTH-003 `[PROPOSED]`), logout, password change invalidation.
- **Exit:** all auth matrices green; throttling verified; no default-credential login possible.

### 6. Authorization Testing
- **Objective:** verify RBAC matrix (`docs/12`) and deny-by-default (`[SOURCE §16]`).
- **Scope:** every role × module × action; permission-matrix runtime changes take effect; privileged actions (matrix edit, exports, integration config) require proper role.
- **Exit:** matrix suite green; any permission not granted = denied; matrix changes audited.

### 7. Farmer Data Isolation Testing
- **Objective:** prove a farmer can access ONLY own data (P-ISO-01…06; F-OD-01…05).
- **API level:** GET /farmer/me/* returns own rows only; no client-supplied farmerId accepted; foreign IDs → 404 (no existence disclosure); ledger/statement/payments scoped.
- **UI level:** no cross-data in dashboard, notifications, statements download.
- **Concurrency:** simultaneous farmers reading same endpoints do not cross-contaminate.
- **DB level:** scoped predicate returns exactly expected rows (TC-05; positive + negative farmers in dataset).
- **Exit:** zero cross-farmer leakage in all run scenarios (SC-05/SC-06).

### 8. Procurement Testing
- **Objective:** verify the purchase workflow (`[SOURCE §5]`, FR-PRO-*).
- **Scope:** 9-step flow, farmer selection (live + Phase 2 in-scope), product/unit validation, qty>0, rate>0, deduction ≤ gross, computed amounts (SC-03), employee attribution, invoice auto-generation, amount preview correctness.
- **Exit:** all validation + amount assertions green.

### 9. Invoice Numbering Testing
- **Objective:** verify numbering scheme: invoice = Farmer ID + per-farmer sequence; unique (`[SOURCE §6]`; Q-DB-05).
- **Scope:** sequence increments per farmer; independent per farmer; no gaps required (Timing/format Q-DB-05); no reuse after cancellation (SC-02); no monthly reset (unless decided).
- **Exit:** uniqueness proven for N farmers × M invoices + cancellation reuse scenario (SC-01/02).

### 10. Invoice Cancellation Testing
- **Objective:** verify cancellation semantics (`[SOURCE §6]`, OQ-08).
- **Scope:** allowed cancel states (pending OQ-08) — cancelled invoice number **never reused** (SC-02); ledger impact of cancellation (posting reversal per `docs/14` correction rules); cancellation audited with before/after; statement reflects cancellation; WhatsApp? (no notification on cancel — Q-NTF-07).
- **Exit:** reuse attempt fails; audit entry present; financial views consistent post-cancellation.

### 11. Payment Testing
- **Objective:** verify payment recording, UTR capture, statuses (`[SOURCE §10, §11]`).
- **Scope:** payment create (mode, bank ref, UTR), statuses Matched/Unmatched/Failed/Pending/Duplicate; UTR uniqueness in ledger; payment → farmer visibility (UTR masked OQ-18); bank/API outbound flow INT-01 (SC-08).
- **Exit:** status lifecycle and UTR mapping correct; no UTR duplication accepted for same payment where schema requires.

### 12. Partial Payment Testing
- **Objective:** verify partial settlement behaviour (Q-PAY-006; allocation model).
- **Scope:** a payment partially satisfying invoice allocations (SC-04); outstanding reduced by allocated amount, not whole invoice; statuses remain consistent (pending remainder); reconciliation registers part-matched.
- **Exit:** allocation amounts sum = payment allocation; outstanding reflects residual; reporting sub-view PM-04 consistent (once model approved).

### 13. Multiple Payment Testing
- **Objective:** verify multiple payments against one obligation (and one payment across multiple invoices).
- **Scope:** two+ payments toward one invoice; allocation ordering; over-allocation guard (docs/09); duplicate-payment detection (`[SOURCE §11]` Duplicate); ledger entries for each payment.
- **Exit:** net allocation = sum payments; over-allocation blocked; ledger shows each with its own UTR.

### 14. Payment Reconciliation Testing
- **Objective:** verify the reconciliation cycle (`[SOURCE §11]`, FR-REC-*).
- **Scope:** UTR matching, bank statement import (INT-01), Unmatched queue (PM-06), resolution actions (retry/match/adjust), status transitions, ledger updating after match, reconciliation events audited.
- **Exit:** matched payments post to ledger; unmatched remain visible in queue; reconciliation notification behaviour per Q-NTF-11.

### 15. Ledger Testing
- **Objective:** verify farmer ledger correctness (`[SOURCE §9]`, FR-LED-*).
- **Scope:** each purchase → debit entry; each payment → credit entry; UTR recorded; ordering by date; derived position per `docs/10` (undefined formulas asserted only on documented dims — Q-LS-*).
- **Exit:** ledger balances consistent with transactions across generated volumes; concurrency-safe posting (`[PROPOSED]` serialization check).

### 16. Outstanding Calculation Testing
- **Objective:** verify outstanding derivation as documented by `docs/10` (formula Undefined → test asserts data mechanics only, not invented values).
- **Scope:** outstanding = purchases - payments (as currently derived in dashboard/reports F-03, K-09); zero/negative/credit-outstanding cases; effect of cancellation and partial payment.
- **Exit:** numbers match recorded-data arithmetic; where formulas remain undefined test is blocked and flagged (Q-TST-04).

### 17. Statement Generation Testing
- **Objective:** verify monthly statement (`[SOURCE §13]`, FR-MST-*).
- **Scope:** 1st-of-month job; snapshot consistency (period frozen); opening from prior closing; purchases/payments of period; PDF generated (INT-03) and dispatched (WhatsApp INT-02); status tracking; retry for undelivered; farmer in-portal copy `[NEW]`.
- **Exit:** statement snapshot matches ledger for the period; PDF valid; delivery states recorded; idempotent re-gen (no number changes).

### 18. WhatsApp Testing
- **Objective:** verify WhatsApp channel (`docs/11`, `Docs 19` INT-02).
- **Scope:** message templates for purchase/payment/statement; PDF attachment; status callbacks (Sent/Delivered/Failed/Retry); provider signing; retry policy F-06; Phase 2 chatbot menu + own-data guard (P-ISO) when implemented.
- **Exit:** callbacks update statuses correctly; no body/phone in logs; retry matices verified (SC-09).

### 19. Notification Retry Testing
- **Objective:** verify notification lifecycle and retry (`docs/20`).
- **Scope:** Created→Queued→Sent→Delivered/Failed→Retry→Dead-letter; per-channel status; in-portal mirror `[NEW]`; retry budgets (Q-NTF-04); no silent drops; no duplicate artefacts on retry.
- **Exit:** matrix (10 in §11 of `docs/20`) green across scenarios incl. provider outage.

### 20. Audit Testing
- **Objective:** verify audit completeness and immutability (`[SOURCE §17]`, `docs/14`).
- **Scope:** all event catalogues (login, procurement, invoice, cancel, payment, reconciliation, farmer, permission, admin); fields actor/action/date/old/new; append-only; no app-path edit/delete; sensitive values masked; log-write failure behaviour (Q-SEC-14).
- **Exit:** audit matrix complete; tamper attempt fails; values accurate for tested mutations (SC-10).

### 21. Security Testing
- **Objective:** verify `docs/18` controls.
- **Scope:** TLS, encryption at rest (`[SOURCE §18]`), session/token hygiene, rate limiting (login/OTP/API), input validation, injection/XSS/CSRF/SSTI, file-upload validation + AV (SC-11), secrets not in logs/repo, webhook signature/replay (SC-08), OTP abuse (Q-SEC-02 dependent).
- **Exit:** OWASP-aligned sweeps (manual + automated scan `[PROPOSED]`); critical/high findings closed before release.

### 22. Performance Testing
- **Objective:** verify responsiveness at documented scale.
- **Scope:** API latency/throughput on typical flows (purchase entry, dashboard KPIs, report generation, statement batch); concurrency of 500 employees (`[SOURCE §1, §19]`); 1st-of-month statement/burst; rate-limit behaviour under load.
- **Exit:** SLA targets defined (Q-TST-06 — NFR doc pending); no regression below baseline.

### 23. Scalability Testing
- **Objective:** prove scale-readiness — 10,000→50,000 farmers, 100→500 employees (`[SOURCE §1, §19]`), million-transaction order (SC-12).
- **Scope:** horizontal scaling of stateless app, DB volume vs indexes/rollups, pre-aggregation (reports `docs/13` §16), storage growth (PDF/KYC), async queue throughput for statements/notifications.
- **Exit:** documented scale passes at target thresholds; growth projections captured Q-TST-20.

### 24. Backup/Restore Testing
- **Objective:** verify `docs/18` §21/§22 (backup/RTO-RPO plan) and data-integrity recovery.
- **Scope:** scheduled full/incremental; encryption at rest; **restore drills on non-prod targets incl. ledger/audit table**; point-in-time recovery; integrity checks; statement/ledger consistency post-restore.
- **Exit:** restored data integrity verified; RTO/RPO targets (once defined) met.

### 25. Regression Testing
- **Objective:** protect existing behaviour across releases.
- **Scope:** curated regression suite from all sections above → CI; suites per release; integration of matrix changes; flaky-tolerance `[PROPOSED]`.
- **Exit:** no open P1/P2 defects (severity scale Q-TST-03) at release.

### 26. UAT
- **Objective:** business sign-off on acceptance (`docs/01` §24 success criteria).
- **Scope:** business-approved scenarios tracing to FR-*; real-ish data set; UAT-only environment; BA/business executors for Admin, Employee, Farmer roles; defect triage during UAT.
  - Eligibility: UAT begins when regression + security gates pass.
- **Exit:** documented acceptance checklist signed; outstanding UAT defects ≤ agreed threshold.

### 27. Special Attention — Detailed Critical Scenarios (Appendix §6)

---

## 6. Detailed Scenarios — Critical Business Rules

Given/When/Then style. Attribute IDs: **SC-01…SC-12**. Assertions assert **documented** behaviour only.

### SC-01 — Invoice sequence uniqueness (never reuse, per-farmer)
| Step | Scenario |
|---|---|
| Given | Farmer F-0001 and Farmer F-0002 exist; F-0001 has 5 invoices (seq 1–5), F-0002 has 2 |
| When | A new purchase is confirmed for F-0001 |
| Then | New invoice = F-0001 + seq 6; F-0002 sequence **unaffected** (remains 2); DB unique index (farmer_id, seq) holds at 6 |
| And | Attempting to insert seq 5 for F-0001 again is rejected (constraint violation) |
| Owner | API + DB level | Requirement | `[SOURCE §6]`, Q-DB-05, `docs/15` TC-01 |

### SC-02 — Cancelled invoice number never reused
| Step | Scenario |
|---|---|
| Given | F-0001 seq 7 cancelled via authorised cancellation flow |
| When | A new purchase is confirmed for F-0001 |
| Then | New invoice gets seq 8 (not 7); seq 7 is the cancelled number only; attempted reuse of 7 rejected |
| And | Cancellation audited (actor, before/after); ledger reversed/annotated per correction contract (`[SOURCE §6]`, `docs/14`) |
| Owner | API + DB | Requirement | `[SOURCE §6]`, OQ-08, TSC-01 regression per `docs/09` |

### SC-03 — Quantity × Rate calculation
| Step | Scenario |
|---|---|
| Given | Purchase: quantity 100 unit, rate X |
| When | Preview/confirm with deduction Y (permitted role) |
| Then | Gross = qty × rate; Net = Gross − Y; server-recomputed (client value never trusted); rounding per number format (Q-RPT-03) |
| And | qty ≤ 0 or rate ≤ 0 rejected; deduction > gross rejected (`docs/09` validation) |
| Owner | Unit + API + UI | Requirement | `[SOURCE §5, §9]`, FR-PRO-* |

### SC-04 — Payment allocation (partial + multiple)
| Step | Scenario |
|---|---|
| Given | Invoice total 10,000; payment 4,000 (partially pays) |
| When | Reconciliation matches 4,000 to the invoice |
| Then | Allocated = 4,000; outstanding reduced by 4,000; remainder stays; status remains consistent (pending residual) (Q-PAY-006 model) |
| And | Second payment 6,000 → total allocations 10,000 = invoice; further allocation blocked (over-allocation guard, `docs/09`) |
| Owner | API + Ledger | Requirement | `[SOURCE §10, §11]`, Q-PAY-006, `docs/13` PM-04 |

### SC-05 — Farmer sees ONLY own data (API)
| Step | Scenario |
|---|---|
| Given | Farmers A and B; A is authenticated; dataset contains records for both |
| When | A calls /farmer/me/ledger, /farmer/me/payments, /farmer/me/statements |
| Then | Only A's rows returned; no A-supplied farmerId used (identity-scoped P-ISO-01) |
| And | A requests B's invoice/statement id directly → **404** (no existence disclosure P-ISO-02); dashboard KPI and notifications show only A |
| Owner | API + UI | Requirement | P-ISO-01…06, F-OD-01…05, DBZ-01, TC-05 |

### SC-06 — Farmer data isolation (concurrent)
| Step | Scenario |
|---|---|
| Given | Farmers A and B authenticated concurrently |
| When | Both poll my-dashboard/ledger simultaneously under load |
| Then | Each response contains only own rows; no cross-session contamination; server scope derived from session |
| Owner | API perf/isol | Requirement | DBZ-01, F-OD |

### SC-07 — Phase 2 employee area restriction (DB-level)
| Step | Scenario |
|---|---|
| Given | Areas 1,2; employee E assigned area 1 only; farmers spread across areas |
| When | E queries farmer list, procurement, reports, exports |
| Then | Every query returns area-1 farmers only — enforced in query/DB predicate (DBZ-02/03), not UI filtering (E-AR-01…03); attempt to fetch area-2 farmer id → denied/404 |
| And | E's report export applies the same area filter (FRPT-02); area transfer re-scopes immediately (E-AR-04) |
| Owner | API + DB | Requirement | `[SOURCE §22, §24]`, E-AR-01…04, DBZ-02/03, FRPT-02 |

### SC-08 — Payment workflow Portal → Bank → UTR/status → Portal
| Step | Scenario |
|---|---|
| Given | Payment approved; provider sandbox |
| When | Portal initiates bank credit; bank processes; UTR/status returns (webhook/poll per Q-PAY-003/Q-INT-03) |
| Then | Payment status transitions Pending → Matched on confirmation; UTR stored (masked display, not logs); ledger credited |
| And | Timeout/no-response → stays Pending, resolved by reconciliation (no assumed-success); duplicate request id rejected (idempotency OQ-13); webhook signature invalid → rejected + audited |
| Owner | Integration/API | Requirement | `[SOURCE §10, §11]`, Q-PAY-003, Q-PAY-005, `docs/19` INT-01, `docs/18` |

### SC-09 — Notification retry (WhatsApp)
| Step | Scenario |
|---|---|
| Given | Statement dispatch fails (provider error) |
| When | Retry policy applied (F-06 `docs/11`; budget Q-NTF-04) |
| Then | Notification: Sent→Failed→(Retry)→Sent/Delivered; after budget → dead-letter for review; in-portal mirror available; no duplicate PDF/statement identity on retry; no body/phone in logs |
| And | Cancelled farmer number → no retry (OQ-06) |
| Owner | Integration/API | Requirement | `[SOURCE §13]`, `docs/11` F-06, `docs/20` §10, OQ-06 |

### SC-10 — Audit trail integrity
| Step | Scenario |
|---|---|
| Given | Invoice created, cancelled; payment reconciled; permission changed |
| When | Audit queried |
| Then | Events present with actor, action, date/time, old/new (masked where sensitive) per `[SOURCE §17]`; append-only — no user path edits/deletes |
| And | Tamper attempt (direct DB write outside app) detectable/blocked per design (`[PROPOSED]` hash-chain per `docs/14`) |
| Owner | DB + API | Requirement | `[SOURCE §17]`, `docs/14`, `docs/18` §20 |

### SC-11 — Upload/export security
| Step | Scenario |
|---|---|
| Given | KYC upload (future §20) and report export |
| When | Files/proforma sent through the boundary |
| Then | Type allow-list + magic-byte verify; size caps; AV scan; stored private; served via signed URLs; export respects role data-scope (FRPT-01…05) |
| And | Path traversal/oversize/executable uploads rejected; scan queue failure does not store unverified file (`docs/18` §19) |
| Owner | Security QA | Requirement | `[SOURCE §18]`, `docs/18` §19, FRPT-* |

### SC-12 — Million-transaction scalability
| Step | Scenario |
|---|---|
| Given | Synthetic dataset of 50,000 farmers, 500 employees; ~1M+ ledger/payment/statement rows (`[SOURCE §1, §19]` order) |
| When | Peak ops run: purchase entry bursts, dashboard KPIs, 1st-of-month statement batch, report generation |
| Then | Query/aggregation path uses rollups/snapshots, pagination, bounded exports — no full-table scans; KPIs and reports within response targets (Q-TST-20/Q-RPT-05); batch jobs complete within window with queue 
backpressure |
| And | No data corruption at volume (ledger balance reconciliation at scale) |
| Owner | Perf/Scalability | Requirement | `[SOURCE §19]`, `docs/13` §16, `docs/15` DI-* |

---

## 7. Test Levels → Requirement Traceability (summary)

| Capability | Test sections |
|---|---|
| Auth (staff/farmer) | 1,3,5,6 |
| RBAC/matrix | 6 |
| Isolation (farmer own-data) | 3,4,7 (SC-05/06) |
| Phase 2 area | 7,1,2,3 (SC-07) |
| Procurement/invoice | 1,2,3,8,9,10 (SC-01…03) |
| Payment/recon/ledger | 1,2,3,11,12,13,14,15,16 (SC-04, SC-08) |
| Statements | 2,15,17 |
| WhatsApp/notifications | 2,3,18,19 (SC-09) |
| Audit | 20 (SC-10) |
| Security/backup | 21,24 (SC-11) |
| Performance/scale | 22,23 (SC-12) |
| Release readiness | 25,26 |

---

## 8. Open Questions & Gating Risks

| ID | Question/Risk | Gate |
|---|---|---|
| Q-TST-01 | Unit coverage gate threshold per module | Confirm before dev commences |
| Q-TST-02 | Which automated tooling (unit/API/UI, mobile viewport runner) | Tool-picking review (no vendor mandated in this doc) |
| Q-TST-03 | Defect severity scale + release thresholds | QA charter sign-off |
| Q-TST-04 | Outstanding formula — tests blocked until `docs/10` decides (Q-LS) | sign-off Q-LS series |
| Q-TST-05 | Payment allocation model (Q-PAY-006) — enables SC-04/PM-04 | sign-off Q-PAY |
| Q-TST-06 | Performance SLAs (NFR doc pending; Q-PRF) | NFR decisions |
| Q-TST-07 | Farmer login method (OQ-02) — auth/OTP tests blocked | auth decision |
| Q-TST-08 | WhatsApp/bank sandbox availability | provider decision Q-INT |
| Q-TST-19 | Invoice number format/zero-pad (Q-DB-05) — numbering assertions | DB review |
| Q-TST-20 | "Millions of transactions" target confirmation (scale fact; source cites 50,000 farmers/500 employees `[SOURCE §1, §19]`) | business confirmation |
| Reused | Q-SEC-14 (audit fail-open/closed), OQ-08 (cancel states), OQ-11 (retention) | prior Q-* cycle |

---

## 9. Test Summary Report Requirements (per release)

- Coverage vs FR-*; defects by severity & module; open risks; performance/scalability numbers vs Q-TST-06 targets; backup/restore drill attestation; UAT sign-off. Trace to `docs/01` §24 success criteria.

---

*End of Testing Strategy v1.0. Next in sequence: `22_GLOSSARY_AND_TERMINOLOGY.md` (optional) or `21_NFR_SCALABILITY` per index.*