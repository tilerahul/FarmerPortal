# Agri Procurement & Farmer Management System
## Open Questions & Assumptions — Master List

---

## 1. Document Information

| Item | Value |
|---|---|
| Product name | Agri Procurement & Farmer Management System |
| Document | Open Questions & Assumptions — Master List |
| Version | v1.0 |
| Status | **Master decision register** — consolidates every open question from all specs (`docs/00…24`); to be clarified before implementation |
| Date | 2026-09-16 |
| Author role | Senior Business & Technical Analyst (consolidation) |
| Purpose | Single list of every unclear/missing/ambiguous/decision-required item: categorised, with owner, options, current assumption where one exists, and impact if unresolved |
| Source | All spec docs: `docs/01…24` (OQ-01…18; Q-PAY, Q-LS, Q-WH, Q-RBAC, Q-RPT, Q-AUD, Q-DB, Q-API, Q-UX, Q-SEC, Q-INT, Q-NTF, Q-TST, Q-DEP, Q-PRF series) |

### How to use this register
- **Canonical rows** = one row per underlying decision, listing **all equivalent Q-IDs** (e.g., `OQ-02 ≡ Q-SEC-02 ≡ Q-API-01`). Resolve each row once; the aliases across docs then close together.
- **Decision owner** is whom the question must go to **before build**.
- "Current assumption" is shown **only where one is already documented** (specs or ASM-*) — never a new assumption introduced here.
- No decisions are taken on the stakeholder's behalf in this document.

### Style
`≡` means the same underlying question. ID prefixes: OQ (original), Q-PAY/Q-LS/Q-WH/Q-RBAC/Q-RPT/Q-AUD/Q-DB/Q-API/Q-UX/Q-SEC/Q-INT/Q-NTF/Q-TST/Q-DEP/Q-PRF (per-doc series).

---

## 2. Category: Business (B-)

| Question ID (≡ aliases) | Question | Why it matters | Affected module | Decision from | Possible options | Current assumption | Impact if unresolved |
|---|---|---|---|---|---|---|---|
| B-01 (OQ-05) | Product master: catalogue, units, rate-entry rules, approval for rate deviations | Drives catalogue UI, procurement entry validation, product reports | Procurement, Products, Reports | Business owner | Manual-code catalogue; configurable list with admin approval; open list + validation | None | Procurement entry ambiguity; reports P-02/P-06 unreliable; UI spec blocked |
| B-02 (OQ-06) | Deduction rules: types (flat/percentage), reasons, approvals, ledger/invoice impact | Every invoice may carry deductions; amount = qty×rate then deduction affects net | Procurement, Invoice, Ledger | Business owner + Finance | None; flat-only; percentage; both w/ approval; no deductions Phase 1 | None | Invoice/ledger calculations unbuildable; SC-03 partially blocked |
| B-03 (OQ-12, CON-005, ASM-007) | Tax/statutory handling: GST, levies, commissions | Legal correctness of invoices/statements | Invoice, Reports, Legal | Finance + Legal | Exclude statutory (Phase 1); include at rates TBD; configurable tax engine | Statutory outside current rules until confirmed; INR base | Incorrect statements/tax exposure; reports invalid |
| B-04 (OQ-09, Q-RBAC-01, Q-RBAC-02, Q-API-10, Q-UX-06) | Full role/permission catalogue; employee access to ledger/reports | RBAC matrix implementation, UI action visibility | RBAC, Reports, UI | Business + Admin | Keep 3 roles (SA/Employee/Farmer); add Accounts role; per-module matrix | Two employee-family roles in PDF; matrix generic `[SOURCE §16]` | Access gaps or over-permission; employee reports blocked |
| B-05 (Q-PAY-010, Q-RBAC-04, Q-DB-10, Q-API-02) | Who creates/approves payments | Payment create/approval path, RBAC rows, API auth | Payment, RBAC, API | Accounts owner | Accounts only; SA only; accounts create + SA approve | Accounts staff implied `[SOURCE §11]` | Payment endpoint exposes risk; audit attribution unclear |
| B-06 (Q-NTF-07) | Notification suppression rules (e.g., cancelled invoice — no notification?) | Avoids confusing/wrong farmer messages | Notifications, Invoice | Business owner | Notify on every state; suppress on cancellation; only confirm positive states | None | Conflicting or missed farmer communications |
| B-07 (Q-PAY-007) | Accepted payment modes (values) | Payment form validation, register columns | Payment, Reports | Accounts | Cash/bank transfer/cheque/UPI set TBD | None | Payment model and PM-01 unsatisfiable at build |

> Business decisions already raised but duplicated elsewhere: B-06→Invoice cancellation; B-07 filtered; see INV-01…03, PAY-01…03.

---

## 3. Category: Farmer (F-)

| Question ID (≡ aliases) | Question | Why it matters | Affected module | Decision from | Possible options | Current assumption | Impact if unresolved |
|---|---|---|---|---|---|---|---|
| F-01 (OQ-01, Q-UX-01) | Farmer Portal delivery phase (Phase 1 vs 2) and exact dashboard widget/feature list | Scope & roadmap locking | Farmer Portal, Roadmap | Product owner | Full portal in P1; subset (ledger/statements) in P1 | Portal is Phase 1 `[NEW]`; feature list open | Portal scope creep or under-delivery |
| F-02 (OQ-13, Q-WH-08, Q-DB-03) | One registered mobile linked to two Farmer IDs — dedup/identity policy | WhatsApp identity anchor ambiguity; security own-data trust | Farmer master, WhatsApp, Auth | Business + Admin | Block duplicates; allow w/ warning; dedup workflow | None; mobile is the identity anchor | Wrong-farmer notifications/statements (privacy risk) |
| F-03 (OQ-15, P-FF-06) | Portal account activation / onboarding flow | How farmers start using the portal | Farmer Portal | Product owner | Admin creates account; farmer self-activates; OTP activation | None | Farmers cannot log in; onboarding unspecified |
| F-04 (OQ-16, Q-UX-09) | Support channel / method for farmers | Support screen cannot be designed | Farmer Portal | Business | Phone helpdesk; WhatsApp support; in-portal form; offline contact | None (support exists as portal requirement FR-PRT-011) | Farmer help path missing; call volume |
| F-05 (OQ-17, Q-UX-03, Q-INT-10, Q-NTF-01) | Portal languages; number/currency display format (currency symbol, decimals, date) | UI strings, formatting, WhatsApp copy | Farmer Portal, WhatsApp, Reports | Business | Hindi/English/multi; INR formats; locale config | INR implied by examples (ASM-001); **not confirmed** | Wrong currency/format; rework of strings; Q-RPT-03 coupling |
| F-06 (OQ-18, Q-RBAC-05, Q-UX-04, Q-SEC-07) | What farmers may view/edit: bank details, KYC; who may reveal; masking policy | Sensitive-data UX and security | Farmer Portal, Security | Business + Compliance | View-only masked; view clear w/ verification; no edit; edit w/ approval | Masked by default for staff; farmer own-view only P-ISO-06 | Privacy/liability; leakage surface |

---

## 4. Category: Procurement (PR-)

| Question ID (≡ aliases) | Question | Why it matters | Affected module | Decision from | Possible options | Current assumption | Impact if unresolved |
|---|---|---|---|---|---|---|---|
| PR-01 (≡ B-01) | Product master & rate rules | see B-01 | Products, Procurement | Business | | | |
| PR-02 (≡ B-02) | Deduction rules | see B-02 | Invoice, Ledger | Business | | | |
| PR-03 (Q-DB-04, DI-15) | Ledger source-post guard: exactly-once posting per invoice / per confirmed payment; duplicate handling | Financial integrity, ledger correctness | Ledger, Database | Data owner + Finance | Unique constraint invoice→entry; payment-one-entry; compensate duplicates | Derived exactly-once rule; enforcement undecided | Double-posting or silent drops at scale |
| PR-04 (Q-NTF-07) | Suppression/notification on cancelled procurement | see B-06 | Notifications | Business | | | |

---

## 5. Category: Invoice (INV-)

| Question ID (≡ aliases) | Question | Why it matters | Affected module | Decision from | Possible options | Current assumption | Impact if unresolved |
|---|---|---|---|---|---|---|---|
| INV-01 (OQ-07, Q-UX-05, Q-API-06) | Invoice output: printable/PDF invoice; reprint rules | Portal/admin print actions, PDF service, export audit | Invoice, Farmer Portal, PDF | Business | Confirmation-as-PDF P1; full invoice PDF incl. itemisation; portal download | Transaction confirmation + monthly statement PDF mandated; invoice PDF open | PDF/portal scope; Q-INT-03 depends |
| INV-02 (OQ-08, Q-RBAC-03, Q-LS-06, Q-AUD-08, Q-INT-08) | Cancellation workflow depth: who may cancel, reason capture, ledger impact, compensation traceability | Cancellation is financially significant | Invoice, RBAC, Ledger, Audit | Business + Finance | Create-cancel only by SA; SA + approvals; reason mandatory; ledger reversal entry | Cancel retains record + number never reused `[SOURCE §6]`; authorization/reason open | Reversal/statement errors; audit gaps |
| INV-03 (Q-DB-05, Q-TST-19) | Invoice sequence reset periodicity (continuous vs monthly) | Number formatting/length; numbering tests | Invoice, Database | Business | Continuous per-farmer; monthly reset; zero-padded fixed width | Per-farmer seq independent `[SOURCE §6]`; reset unknown | Numbering assertions in tests blocked; format drift |

---

## 6. Category: Payment (PAY-)

| Question ID (≡ aliases) | Question | Why it matters | Affected module | Decision from | Possible options | Current assumption | Impact if unresolved |
|---|---|---|---|---|---|---|---|
| PAY-01 (Q-PAY-006, Q-RPT-09, Q-TST-05, PM-04) | Allocation split rule (FIFO etc.) and over-allocation guard | Partial/multiple payments; outstanding correctness | Payment, Ledger, Reports | Finance | FIFO (oldest invoice); manual allocation; pro-rata; guard block/soft | Source supports full/partial/multiple `[SOURCE §10]`; guard proposed; rule undefined | PM-04/SC-04 unreproducible; reconciliation ambiguous |
| PAY-02 (Q-PAY-002) | Payment sync mechanism, frequency, reconciliation window | Recon scheduling; statement timing | Payment, Reconciliation | Accounts + Banking | Poll interval; webhook; daily statement import; window (T+?) | Statuses come from bank where supported `[SOURCE §11]` | Recon queue never settles correctly |
| PAY-03 (Q-PAY-003, Q-AUD-03) | Reconciliation queue states, workflow, escalation; who reopens/corrects closed cases | Worklist UI, resolution actions | Reconciliation, Audit | Accounts | States per §11 only; +escalation states; SA-only reopen | §11 statuses mandated; workflow undefined | Queue logic unbuildable; audit correction gap |
| PAY-04 (Q-PAY-004) | Retry/re-initiation of failed payments; reversal procedure for duplicates | Money safety around retries | Payment, Reconciliation | Finance | Auto-retry idempotent; manual from queue; reversal via new entry | No silent auto-re-credit (design) | Duplicate/deficit credits |
| PAY-05 (Q-PAY-009) | Pending-payment time-to-outcome / expiry | When Pending resolves to Failed/Unmatched | Payment | Accounts | Hold N days then re-reconcile; escalate after N; no expiry | None | Pending statuses linger |

---

## 7. Category: Banking (BNK-)

| Question ID (≡ aliases) | Question | Why it matters | Affected module | Decision from | Possible options | Current assumption | Impact if unresolved |
|---|---|---|---|---|---|---|---|
| BNK-01 (OQ-03, Q-PAY-001, Q-INT-03, Q-INT-04) | Bank/API provider and integration mode (push/pull; webhook vs polling; UTR availability timing) | The mandated payment workflow depends on it | Banking, Payment, Integrations | Business + Banking partner | REST push + webhook; SFTP statement pull; polling; provider-specific | Desired workflow Portal→Bank→Payment→UTR→Portal `[SOURCE §10, §11]` | Payment statuses unreliable and integration unbuildable |
| BNK-02 (Q-INT-08) | Bank statement file format & ingestion schedule | Reconciliation batch | Reconciliation | Banking partner | CSV/XML; daily/end-of-day | None | Unable to reconcile |
| BNK-03 (Q-API-04, Q-SEC-10) | Webhook credential/signing model + replay protection; partner TLS termination | Security of inbound status | Banking, Security | Security + Banking | Signed payload; mTLS; IP allow-list | Signatures required; scheme open | Forged status updates |
| BNK-04 (Q-INT-05, Q-PAY-004) | Timeout and retry budgets for bank API | Pending-state fidelity, load | Banking | Accounts + DevOps | e.g., 30s/3 retries (PROPOSED) | Not claimed | Infinite retries or premature failure |
| BNK-05 (Q-INT-06) | Bank rate/volume ceilings and burst policy | Payment runs throttling | Banking | Banking partner | Published ceilings; queueing | None | Payment blasts blocked |
| BNK-06 (Q-INT-07) | Fallback: is manual payment entry/status adjustment always allowed | Failure-mode ops | Payment, Reconciliation | Finance | Yes (recon fallback); only with audit | Portal authoritative; manual fallback undefined (Q-PAY-005) | Operations stall during bank outage |
| BNK-07 (Q-INT-15, Q-SEC-20) | Bank-vendor DPA / data-residency / contractual security obligations | Legal/security posture | All, Legal | Procurement + Legal | Accept provider DPA; negotiate bespoke | **Nothing claimed** | Compliance exposure |

---

## 8. Category: WhatsApp (W-)

| Question ID (≡ aliases) | Question | Why it matters | Affected module | Decision from | Possible options | Current assumption | Impact if unresolved |
|---|---|---|---|---|---|---|---|
| W-01 (OQ-04, Q-WH-01, Q-INT-09) | WhatsApp provider & message policy (utility vs template; approval/cost); opt-in semantics | Every notification depends on this | WhatsApp, Notifications | Business + provider | Meta Cloud API; third-party gateway; template vs utility | WhatsApp channel mandated `[SOURCE §8, §12, §13]`; provider/mode open | Notifications cannot be built |
| W-02 (Q-WH-02, Q-NTF-01) | Exact message template copy/tokens for purchase, payment, statement | Copy, tokens, localisation | WhatsApp, Notifications | Business | Publish templates per F-05 language; indicative text from `[SOURCE §8, §12]` | Indicative text only | Templates not approvable |
| W-03 (Q-WH-03, Q-INT-11, Q-PAY-008) | Delivery-status tracking & failure handling for purchase/payment (statuses mandated only for statements) | Retry UI, undelivered handling | WhatsApp, Notifications | Business | Track everything; track statements only; fallback text | Statement delivery tracked; purchase/payment open | Farmer not knowing failure; UX confusion |
| W-04 (Q-WH-04, Q-NTF-04, Q-LS-03) | Retry policy: frequency, attempts, backoff, manual override (notifications, statements, delivery) | Notification state machine | Notifications, Statement | Business + Ops | e.g., 3 retries/backoff then manual; per-type budgets | Statement retry implied `[SOURCE §13]`; numbers open | Silent loss or unbounded retries |
| W-05 (Q-NTF-06, Q-WH-03) | Statement "undelivered" semantics & farmer action (download alternative) | Farmer access to statement if WhatsApp fails | Statement, Farmer Portal | Business | In-portal always; text-with-link; SMS fallback | Statement PDF via WhatsApp `[SOURCE §13]` | Farmer never sees statement |
| W-06 (Q-NTF-03, P-FF-02) | In-portal read/receipt state required | Notification UX | Farmer Portal | Product | Unread badge + read; none | None | Minor UX, but UI spec pending |
| W-07 (Q-NTF-02, Q-UX-08) | In-portal notification retention window | History length | Farmer Portal | Business | e.g., 90 days; keep all; OQ-11 aligned | None | UI/DB list design blocked |
| W-08 (Q-NTF-05) | Notification volume/lifecycle limits per farmer (spam guard) | Provider rate ceilings, UX | Notifications | Business + Ops | Cap/day; coalescing | None | Provider bans; noise |
| W-09 (Q-NTF-08) | SMS fallback channel approval | Channel beyond source | Notifications | Business | Approve SMS fallback; not | No SMS assumed | Fallback path undefined |
| W-10 (Q-NTF-09) | Admin alert channels (email/SMS) approval | Ops alerting | Notifications, Ops | Ops | Email alerts; none | No email assumed | Admin unaware of failures |
| W-11 (Q-NTF-11, Q-PAY-008) | Dedup rules when reconciliation changes an already-notified payment | Conflicting farmer messages | Notifications | Business | One notification per final state; suppress interim | None | Duplicate/contradictory messages |

---

## 9. Category: Authentication (A-)

| Question ID (≡ aliases) | Question | Why it matters | Affected module | Decision from | Possible options | Current assumption | Impact if unresolved |
|---|---|---|---|---|---|---|---|
| A-01 (OQ-02, Q-SEC-02, Q-API-01, Q-NTF-10, Q-TST-07) | Farmer authentication method | Security vs simplicity; OTP endpoints | Farmer Login, Auth, Security | Product + Security | OTP-on-mobile (recommended anchor); credentials; both | Mobile is identity anchor `[SOURCE §3, §21]`; OTP is `[PROPOSED]` | Farmer portal cannot be built/tested |
| A-02 (Q-SEC-03) | OTP fallback when farmer loses/OTP-registered mobile | Locked-out farmers | Farmer Login | Business + Security | Admin override; alternate contact; offline verification | None | Farmer account recovery gap |
| A-03 (Q-SEC-01) | Password parameters (length/complexity/expiry) for staff | `[SOURCE §18]` "strong password" needs specificity | Auth, Security | Security | e.g., ≥12/mixed/90d; None | "Strong" required; params open | Policy unenforceable |
| A-04 (Q-SEC-04, FR-AUTH-002) | Employee password self-service/reset flow | Helpdesk load; security | Auth | Business + Security | Self-service w/ admin reset; admin-only reset | Login + change via A1/E7 exists | Staff lockout handling |
| A-05 (Q-SEC-05) | Admin password reset authority & break-glass path | Privileged access continuity | Auth, Security | Security | SA-only; emergency break-glass | Privileged accounts restricted `[SOURCE §2]` | Admin lockout = outage |
| A-06 (Q-SEC-06, Q-API-03, FR-AUTH-005) | Session timeout per role; access/refresh token model & expiry | Session risk, farmer convenience | Auth, API, Security | Security + Business | e.g., admin 8h/idle; farmer longer; short access token + rotating refresh | Roles defined; token model (access+refresh) exists in API spec as design | Session theft or logouts |
| A-07 (Q-SEC-12) | Concurrent farmer sessions allowed | Multiple devices/security | Auth | Business + Security | Allow 1; allow N limited | None | UX or abuse risk |

---

## 10. Category: Security (S-)

| Question ID (≡ aliases) | Question | Why it matters | Affected module | Decision from | Possible options | Current assumption | Impact if unresolved |
|---|---|---|---|---|---|---|---|
| S-01 (Q-SEC-07, OQ-18) | Must every KYC view/access be logged | Privacy stance | KYC, Audit | Security + Compliance | Log every access; log downloads only | Masking by default (OQ-18-related) | Privacy gap |
| S-02 (Q-SEC-08) | KYC retention period | Legal compliance | KYC, Storage | Compliance | Tied to relationship; fixed term | None | Storage/compliance exposure |
| S-03 (Q-SEC-09) | Mandatory TLS formalisation (policy or de-facto) | Transport guarantee | Infrastructure, Security | Security | Formal policy; always-on TLS | TLS ≥1.2 proposed | Weak transport |
| S-04 (Q-SEC-10 ≡ BNK-03) | Partner TLS termination (bank/WhatsApp) | Trust boundary | Integrations, Security | Security | Terminate at partner using their cert; verify | None | Weak transport security at boundary |
| S-05 (Q-SEC-11) | Sanitised (masked) non-production backups required | Data-leak safeguards | Backup, DevOps | Security + Ops | Masked copies; none | None | Sensitive data in non-prod |
| S-06 (Q-SEC-13) | Upload AV scanning vendor/policy | Malware hygiene | Storage, KYC | Security + Ops | Scan-before-store; quarantines | AV required proposed | Stored malware |
| S-07 (Q-SEC-14) | Audit-log write failure: fail-open vs fail-closed | Availability vs audit guarantee | Audit, Infrastructure | Security + Business | Fail-closed (block op); fail-open (log-and-warn) | None | Fatal if wrong — audit guarantee lost either way |
| S-08 (Q-SEC-15) | Backup RPO / frequency | Data-loss ceiling | Backup | Business + Ops | e.g., daily/PITR hourly | None | Unacceptable data loss window |
| S-09 (Q-SEC-16) | Backup RTO / restore target | Downtime ceiling | Backup, DR | Business + Ops | e.g., 24h RTO staging drill | None | Extended outage |
| S-10 (Q-SEC-17) | Incident-response ownership & escalation | Operation during breaches | Ops | Business + Security | Named owners/escalation; none | None | Confusion during incident |
| S-11 (OQ-11, Q-SEC-18, Q-AUD-01, Q-DB-06, Q-INT-18, Q-NTF-02, Q-UX-08, Q-PRF-10) | Data/audit retention windows and archival/export format | Storage growth, compliance | Database, Audit, Storage | Compliance + Business | Retention per data class; archive format; purge | None anywhere | Storage bloat; legal exposure |
| S-12 (Q-AUD-02) | Audit: request/response body logging vs field-level only | Log size vs forensics | Audit | Security | Field-level (default); body for auth? | Field-level default proposed | Incomplete forensics or log bloat |
| S-13 (Q-AUD-03) | Who may reopen/correct a closed reconciliation case | Audit integrity | Reconciliation, Audit | Accounts + Security | SA-only; accounts w/ audit | None | Undetected corrections |
| S-14 (Q-AUD-04) | System-actor identity for scheduled jobs (statement/WhatsApp) | Attribution | Audit | Business + Security | Named service account per job; "SYSTEM" | None | Unattributable batches |
| S-15 (Q-AUD-05, Q-API-06) | Report view/export audit granularity (which views logged) | Audit completeness vs volume | Reports, Audit | Security + Ops | Log exports+privileged views; only exports | Exports logged (proposed) | Missing audit evidence |
| S-16 (Q-AUD-06) | Audit-log backup/DR obligations | Legal evidence durability | Audit, DR | Security + Compliance | Audit in DR plan; separate ledger | None | Evidence loss on disaster |
| S-17 (Q-AUD-07) | Should WhatsApp delivery status trace into ledger/statement? | Delivery vs financial state semantics | WhatsApp, Ledger | Finance + Security | Delivery ≠ finance; keep separate | Statuses separate from ledger by design | Confusion over "delivered" vs "paid" |

---

## 11. Category: Database (DB-)

| Question ID (≡ aliases) | Question | Why it matters | Affected module | Decision from | Possible options | Current assumption | Impact if unresolved |
|---|---|---|---|---|---|---|---|
| DB-01 (Q-LS-01, Q-DB-01, Q-TST-04) | Exact opening/closing balance and outstanding computation rule + rounding/precision policy | Statement and outstanding correctness | Ledger, Statement, Reports | Finance | Historical-only; include invoices/paid; formula per `docs/10` | Outstanding = purchases − payments (derived); exact rule undefined | F-03/K-09/SC-16 & statements blocked |
| DB-02 (Q-DB-02) | Decimal scale/rounding for money and rates | Consistent totals | All financial | Finance | 2dp mid-rule; scalars, tie-break | Default chosen in `docs/15`; confirm | Off-by-paise differences |
| DB-03 (Q-DB-08) | Indexes: near-real-time outstanding vs nightly materialisation | Query cost vs freshness | Dashboard, Ledger | Data + Ops | Real-time aggregated; nightly rollups | Rollup strategy recommended | K-09/F-03 latency or stale |
| DB-04 (Q-DB-09, Q-API-08) | timezone/timestamp policy (dates stored/served; ISO offsets on filters) | Statement "1st", date filters, audit times | Database, API | Ops + Business | UTC storage + local display; server-local | None | Statement day boundary errors |
| DB-05 (Q-DB-06) | Audit retention/archival + append-only mechanism (hashing/WORM) | See S-11; append-only enforcement | Audit, Database | Security + Ops | Hash-chain; WORM storage; plain append-only | Append-only in app design; mechanism open | Audit tamper risk |
| DB-06 (Q-DB-07, Q-RBAC-06, Q-UX-10) | Area modelling granularity: multiple areas per employee; hierarchy? | Phase 2 scope model | Database, RBAC, UI | Business + Data | 1:N; area tree; farmer↔areas | Phase 2 areas `[SOURCE §22]`; model open | Phase 2 DB/authorization design blocked |
| DB-07 (Q-DB-03) | Mobile dedup/unique-key policy on FARMER.mobile | See F-02 | Database | Business | | | |
| DB-08 (Q-DB-11) | Database technology / storage engine selection (relational vendor etc.) | All deployment/ops sizing | Database, DevOps | Technical lead | Relational options; on-prem vs cloud DB | Logical model only in `docs/15`; none chosen | CI/CD, backups, perf testing blocked |

> Note: `Q-DB-11` is introduced here to name the DB-vendor decision (previously mis-referenced as Q-DB-01 in `docs/22/23/24`; doc 15's Q-DB-01 is the outstanding rule above).

---

## 12. Category: Reporting (RPT-)

| Question ID (≡ aliases) | Question | Why it matters | Affected module | Decision from | Possible options | Current assumption | Impact if unresolved |
|---|---|---|---|---|---|---|---|
| RPT-01 (Q-RPT-01, Q-INT-14) | Exact report layouts/columns for R-01…R-09 | Report production | Reports | Business + Ops | Column sets per `docs/13` F/P/PM groups; PDF template | Suggested columns in `docs/13`; exact layout open | Report deliverable blocked |
| RPT-02 (Q-RPT-02) | Schedules, drill-down, saved filters, comparative periods | Advanced reporting backlog | Reports | Business | None Phase 1; scheduled later | Not in scope by default | If treated required, scope creep |
| RPT-03 (Q-RPT-03) | Export format details: currency, date, rounding, Excel sheets | Export correctness | Reports, Exports | Business + Ops | Sheets-per-group; formats per F-05 | PDF/Excel mandated `[SOURCE §14]`; CSV proposed | Export inconsistencies |
| RPT-04 (Q-RPT-04) | Email/WhatsApp distribution of reports | Distribution channel | Reports, Notifications | Business | Manual download; scheduled distribution | Manual download only | Automation ambiguity |
| RPT-05 (Q-RPT-05) | Dashboard refresh frequency & drill-through | Staleness vs load | Dashboard | Business + Ops | e.g., near-real-time daily today; periodic counts | Periodic for counts (docs/13) | Dashboard uncertainty |
| RPT-06 (Q-RPT-06) | KPI value vs count semantics (K-04…K-08) | KPI meaning | Dashboard | Business | Count-only; value+count | Count by default | Wrong KPI cards built |
| RPT-07 (Q-RPT-07) | Approve CSV export | Format not in source | Reports, Exports | Business | Add CSV; keep PDF/Excel only | Not in source; proposed | Batch/recon pain, or scope surprise |
| RPT-08 (Q-RPT-08) | Approve P-05/P-06 quantity/rate-wise views | Depth of procurement analytics | Reports | Business | Include; exclude | Proposed additions | Report breadth decided late |
| RPT-10 (Q-RPT-10) | As-of outstanding report windowing & snapshot policy | Outstanding reports consistency | Reports | Finance | As-of snapshot; period-live | Fees derived | Conflicting outstanding numbers |

---

## 13. Category: UI/UX (UX-)

| Question ID (≡ aliases) | Question | Why it matters | Affected module | Decision from | Possible options | Current assumption | Impact if unresolved |
|---|---|---|---|---|---|---|---|
| UX-01 (Q-UX-07) | Employee "My Day"/home screen required | Employee landing page | Employee Portal | Product | Rich home; simple nav-to-task | None (proposed lightweight) | Employee UI scope |
| UX-02 (Q-UX-02) | Farmer login UI (OTP vs credentials) | see A-01 | Farmer Login | Product + Security | | | |
| UX-03 (Q-UX-10) | Area/manager assignment UX (Phase 2) | Phase 2 admin flows | Admin, Area | Product | Wizards; bulk assign | None | Phase 2 admin UI blocked |

> UX-01/02/03 aliases resolve in other categories (F-01, A-01, DB-06).

---

## 14. Category: Infrastructure (INF-)

| Question ID | Question | Why it matters | Affected module | Decision from | Possible options | Current assumption | Impact if unresolved |
|---|---|---|---|---|---|---|---|
| INF-01 (Q-DEP-01) | Migration strategy (additive-first; backward-compat window) | Release safety | DevOps | Tech lead | Additive migrations; compat window | None | Downtime/rollback risk |
| INF-02 (Q-DEP-02) | CI/CD tooling selection | Pipeline | DevOps | Tech lead | Various providers | None mandated | Pipeline blocked |
| INF-03 (Q-DEP-03) | Hosting provider / on-prem posture + regions | All deployment | Infrastructure | Business + Tech | Cloud, on-prem, hybrid | None | Deployment architecture unexecutable |
| INF-04 (Q-DEP-04) | DR tier (secondary site/zone vs off-site backups) | RTO/RPO attainment | DR | Business + Ops | Secondary site; backup-only | None | See S-08/09 |
| INF-05 (Q-DEP-05) | Queue/worker technology | Background jobs | Workers | Tech lead | Options per provider | None | Statements/jobs blocked |
| INF-06 (Q-DEP-06, Q-INT-16/17) | Storage service + Phase 1 need for document storage | KYC/PDF storage | Storage | Tech + Business | Object store; DB BLOB; not in P1 | KYC is future `[SOURCE §20]`; storage "if required" | Storage architecture blocked |
| INF-07 (Q-DEP-07) | Log/monitor/error-tracking stack | Observability | Ops | Tech + Ops | Various stacks | None | Ops blind |
| INF-08 (Q-DEP-08) | Container/runtime & orchestrator | App deployment | DevOps | Tech lead | Choices | None | Portability |

---

## 15. Category: Scalability & Performance (PERF-)

| Question ID | Question | Why it matters | Affected module | Decision from | Possible options | Current assumption | Impact if unresolved |
|---|---|---|---|---|---|---|---|
| PERF-01 (Q-PRF-01) | Cache technology choice | Read performance | API, Dashboard | Tech lead | In-memory; distributed cache | None | Dashboard latency |
| PERF-02 (Q-PRF-02, Q-TST-06) | Response-time/throughput budgets per tier (SLAs) | Acceptance & testing targets | All | Business + Tech | P95 budgets per module | None — not claimed | Performance testing lacks targets |
| PERF-03 (Q-PRF-03) | Metric retention/logging volume scope | Storage/ops cost | Ops | Tech + Ops | TTLs per metric class | None | Log growth |
| PERF-04 (Q-PRF-04) | Capacity review cadence/trigger | Growth readiness | Ops | Tech lead | Per release; on scale trigger | None | Late scaling |
| PERF-05 (Q-PRF-05) | Concurrent employee profile at peak (500 active?) | Sizing workers/nodes | DevOps | Business + Tech | e.g., 500 concurrent; login overlap | 500 employees total `[SOURCE §19]`; concurrency unknown | Wrong sizing |
| PERF-06 (Q-PRF-07) | Statement burst throughput/pacing targets | 1st-of-month jobs | Workers | Tech + Ops | e.g., statements/hour caps | None | Burst overload |
| PERF-07 (Q-PRF-08) | Seasonal peak multiples in capacity model | Over-allocation | Capacity | Business | e.g., 2× peak season | None | Under-provisioning |
| PERF-08 (Q-TST-20) | Confirm "millions of transaction records" target (scale fact) | Test dataset size | Perf tests | Business | 1M+ ledger/audit; confirm row count | User-stated target; not §19 figure | Perf evidence mismatch |

---

## 16. Category: Legal / Privacy (LP-)

| Question ID (≡ aliases) | Question | Why it matters | Affected module | Decision from | Possible options | Current assumption | Impact if unresolved |
|---|---|---|---|---|---|---|---|
| LP-01 (Q-SEC-19) | Which regulators/standards/acts the business intends to comply with | Determines requirement set | All | Legal + Business | e.g., Data Protection act, banking stds | None — nothing claimed | Compliance cannot be engineered |
| LP-02 (Q-SEC-20) | Legal verification: privacy, banking, telecom/channel compliance | Contractual/legal exposure | All | Legal | Counsel review per jurisdiction; DPA sign-offs | **No compliance claimed** | Legal exposure |
| LP-03 (ASM-003) | Farmer consent/opt-in to WhatsApp/messages | Sending legitimacy | WhatsApp | Legal + Business | Opt-in capture at registration; confirm | Consent assumed (ASM-003) — unverified | Regulatory complaint risk |
| LP-04 (Q-INT-15 ≡ BNK-07) | Provider DPA/data-residency obligations | see BNK-07 | All | Legal | | | |
| LP-05 (Q-SEC-07/08, OQ-18, S-01/02) | KYC privacy posture (access logging, retention, farmer edit rights) | see S-01/02, F-06 | KYC, Legal | Legal + Security | | | |

---

## 17. Category: Phase 2 (P2-)

| Question ID (≡ aliases) | Question | Why it matters | Affected module | Decision from | Possible options | Current assumption | Impact if unresolved |
|---|---|---|---|---|---|---|---|
| P2-01 (Q-WH-05, Q-API-09) | Chatbot conversation design, response format, unhandled-query behaviour, API contract | Phase 2 scope definition | Chatbot, API | Product | Command-based menu; free-text intent | 9 commands from `[SOURCE §21]` | Chatbot unbuildable |
| P2-02 (Q-WH-06) | Identity verification depth for inbound sensitive queries (OTP/PIN selection) | Same mobile → phishing risk | Chatbot, Security | Security + Business | Mobile-only (source); +PIN/OTP for sensitive | Identified by mobile `[SOURCE §21]` | Account-takeover via phone number |
| P2-03 (Q-WH-07, S-15-adj | Audit/retention of message & chatbot events | Phase 2 accountability | Audit | Security + Ops | Chat history retention; masked | None | Phase 2 audit gap |
| P2-04 (Q-DB-07 / Q-RBAC-06, DB-06) | Area granularity model | see DB-06 | Area, RBAC | Business + Data | | | |
| P2-05 (Q-PRF-09) | Phase 2 WhatsApp chat concurrency/lanes | Chat scale | WhatsApp | Tech + Ops | Dedicated lanes | None | Chatbackpressure |
| P2-06 (Q-UX-10) | Area assignment UX | see UX-03 | Admin | Product | | | |

---

## 18. Assumptions Currently Documented (ASM)

From `docs/01` (unchanged — listed here for the register):

| ID | Assumption | Status |
|---|---|---|
| ASM-001 | Business operates in INR (₹); source examples use ₹ | Confirm (ties F-05, RPT-03) |
| ASM-002 | Procurement employees have internet-connected devices at collection points | Validate with ops |
| ASM-003 | Farmers receiving WhatsApp have opted in / consented | Legal review (LP-03) |
| ASM-004 | Bank/API provider exposes payment status and UTR programmatically, fully or partially (`[SOURCE §11]` "wherever supported") | Provider decision (BNK-01) |
| ASM-005 | Farmer portal users have a mobile/web browser; responsive, not native app (N-03) | Accepted `[NEW]` |
| ASM-006 | A valid WhatsApp number is a precondition for notifications; failed numbers surface in statuses | Behavioural, accepted |
| ASM-007 | Statutory details (GST, levies) out of business rules until confirmed (OQ-12) | Ties B-03 |
| CON-004 | Database must handle millions of transaction records | Confirm target (PERF-08) |

---

## 19. Cross-Document Reference Corrections (applied)

Corrected while consolidating (kept for transparency):

| Doc | Change |
|---|---|
| `docs/13` | Q-PAY-005 → **Q-PAY-006** for allocation model (4 occurrences) |
| `docs/21` | Q-PAY-005 → **Q-PAY-006** for allocation model (4 occurrences; reconciliation ref kept) |
| `docs/24` | Q-PAY-005 → **Q-PAY-006** for allocation model (3 occurrences) |
| `docs/16` | Dropped dangling `OQ-19` (not defined anywhere) from Q-API-02; kept Q-PAY-010 |
| `docs/14` | Q-AUD-01 origin `(OQ-02)` → `(OQ-11)` (retention — was a typo) |
| Q-TST numbering | `docs/21` uses Q-TST-01…08 then 19–20; **Q-TST-09…18 are unused/reserved** |
| Q-DB-01 ambiguity | `docs/22/23/24` referred to "Q-DB-01 (DB selection)"; doc 15's Q-DB-01 is the outstanding rule (Q-LS-01). DB-vendor decision is now **DB-08 (Q-DB-11)** in this register; consider adding Q-DB-11 to `docs/15` |

---

## 20. Decision Cycle Recommendation

- Resolve in order: **A-01 (auth), B-01/02/03 (product/deduction/tax), PAY-01 (allocation), Q-LS-01 (outstanding rule), BNK-01/W-01 (providers), OQ-09 (roles), OQ-11 (retention), Q-SEC-15/16 (RPO/RTO), Q-DB-11 (DB), INF-03 (hosting)** — these unblock most other questions and all build tracks.
- Decisions should be recorded back in this register with status/resolution as they are made.

---

*End of Open Questions & Assumptions — Master List v1.0.*