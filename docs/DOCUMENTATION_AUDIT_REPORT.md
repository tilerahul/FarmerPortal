# Documentation Audit Report

**System:** Agri Procurement & Farmer Management System
**Scope:** Complete cross-document consistency audit of `AgriProcurement & Farmer Management.pdf` (source of truth) vs. all documents in `docs/` (00–25).
**Status:** Read-only audit. No existing document was modified in this step.
**Date:** 17 Sep 2026

---

## 1. Ground Truth & Method

- **Source:** `AgriProcurement & Farmer Management.pdf` — 6 pages, sections §1–§24.
- **Extraction note:** The PDF is glyph/CID-encoded; text was extracted via a CLI PDF tool (`pypdf`). The authoritative extracted text was used as ground truth (`§1 Project Objective … §24 Phase 2 Reporting`).
- **Source integrity fact:** The PDF **ends mid-sentence** with the orphan phrase *"Corrections and cancellations"* — i.e., the source itself truncates and gives **no requirements for corrections/cancellations**. Any correction/cancel rule in the docs is therefore either derived from §17/§6 or `[NEW]`/`[PROPOSED]`, and must not be tagged `[SOURCE §n]` for content the PDF does not contain.
- **Source integrity fact:** The PDF defines **no farmer login** and **no mobile app for farmers in Phase 1** (§1). The Farmer Portal with Farmer Login is a **newly approved requirement**, documented as `[NEW]` (docs/07:20). Any doc implying a farmer login is PDF-mandated is wrong.
- **Tagging convention audited:** `[SOURCE §n]` = stated in PDF; `[NEW]` = Farmer Portal addition; `[PROPOSED]` = proposal needing approval; undefined/open question = gap.
- Every finding cites exact `file:line` evidence. Only genuine, substantive issues are reported; trivial PDF-extraction formatting differences were ignored.

---

## 2. Executive Summary

| Severity | Count |
|---|--:|
| CRITICAL | 3 |
| HIGH | 8 |
| MEDIUM | 16 |
| LOW | 32 |
| INFO | 3 |

The documentation set is **substantively faithful** to the PDF (see §7 Verified-OK Areas). The most damaging issues are (a) three **CRITICAL** misclassifications in `docs/18` where PDF-confirmed §18 security requirements are downgraded to `[PROPOSED]`; (b) **HIGH** misclassification of the §19 "millions of transaction records" requirement as a user-stated target, of §14 export/formats as proposals, an invented "resets monthly" invoice rule, and three **open-question ID collisions** (Q-PAY-003, Q-DB-01, OQ-13) that corrupt cross-document decision tracing; and (c) a **stale Documentation Index + broken navigation footers** that point to non-existent files.

---

## 3. Findings

### 3.1 CRITICAL

| ID | Document | Section | Problem | Source requirement | Impact | Recommended resolution |
|---|---|---|---|---|---|---|
| AUD-001 | `18_SECURITY_AND_PRIVACY.md:173` (also :41) | 13. Encryption in Transit / standing table | "explicit in-transit clause is not present in the captured source — transport hardening is [PROPOSED]" — false. | §18: "Encryption in transit" | Confirmed TLS/transport requirement may be de-scoped at build. | Mark transport encryption as `[SOURCE §18]`; only TLS version/HSTS specifics stay `[PROPOSED]`. |
| AUD-002 | `18_SECURITY_AND_PRIVACY.md:268` | 21. Database Backups | Scheduled backups labelled `[PROPOSED]` "(not explicitly in the captured PDF text … PDF only states encrypted storage §18)". | §18: "Regular automated database backups" | Mandatory backup requirement treated as optional. | Record backups as `[SOURCE §18]`; only RPO/frequency remain open (Q-SEC-15/16). |
| AUD-003 | `18_SECURITY_AND_PRIVACY.md:278` | 22. Backup Restoration Testing | Restore testing labelled `[PROPOSED]` "(PDF has no statement)". | §18: "Backup restoration testing" | Restore-drill guarantee weakened; audit traceability gap. | Mark restore testing as `[SOURCE §18]`; drill schedule/RTO-RPO remain open. |

### 3.2 HIGH

| ID | Document | Section | Problem | Source requirement | Impact | Recommended resolution |
|---|---|---|---|---|---|---|
| AUD-004 | `18_SECURITY_AND_PRIVACY.md:360` | 28. Source Attribution Notes | "Everything else (… backups, restore testing …) is `[PROPOSED]`" perpetuates AUD-002/003 in the doc's own summary. | §18 backups/restore-testing bullets | Reviewers see a false "proposed" classification. | Remove backups/restore-testing (and sessions, rate limits) from the "everything else [PROPOSED]" list. |
| AUD-005 | `23_SCALABILITY_AND_PERFORMANCE.md:28`; `25_OPEN_QUESTIONS_AND_ASSUMPTIONS.md:233` (PERF-08/Q-TST-20) | §1 Transaction density | "potentially millions of transaction records — user-stated design target … not an explicit §19 figure" downgrades a confirmed source requirement to an open question; contradicts `docs/01:411`, `docs/00:100`, `docs/15:811`. | §19: "Database must support potentially millions of transaction records." | Perf/scalability work gated on a confirmation the source already gives; risk of silent drop. | Cite `[SOURCE §19]` in docs/23:28; reframe PERF-08/Q-TST-20 to "confirm dataset row profile", not existence of the requirement. |
| AUD-006 | `14_AUDIT_LOGGING_AND_DATA_INTEGRITY.md:49,217` | 3. Integrity Sources / 10. Data Integrity | "Unique invoice numbers (… no duplicates, **resets monthly**)" tagged `[SOURCE §6]`. The PDF defines a **continuous, per-farmer sequence** (F-0001-01 … F-0001-17) with no monthly reset. | §6: "Sequence is maintained separately for every farmer." | An invented "monthly reset" contradicts the numbering model and would break uniqueness/sequence integrity if implemented. | Remove "resets monthly" or tag `[NEW]`/open question; cite only "unique per-farmer sequence, no reuse". |
| AUD-007 | `13_REPORTS_AND_DASHBOARD_SPECIFICATION.md:181,193,205,217,229,247,259,271,283,295,307,325,337,349,361,373,385,397,543` | §9 Export Formats / report rows | CSV is tagged `[PROPOSED]` "Not stated in source" (Q-RPT-07) in ~17 rows; §19 also silently drops CSV ("PDF/Excel export"). | §14: "Export Excel, PDF and CSV where appropriate" | Confirmed §14 export format may be de-scoped. | Relabel CSV as `[SOURCE §14]` everywhere; delete/repurpose Q-RPT-07; restore CSV on line 543. |
| AUD-008 | `13_REPORTS_AND_DASHBOARD_SPECIFICATION.md:287,290,299,302` | 7. Procurement Reports (P-05, P-06) | Quantity-wise and rate-wise reports tagged `[PROPOSED]` "not in §14 list". | §14: "Procurement … quantity-wise and rate-wise" | Two mandated report categories risk being dropped/re-approved. | Remove `[PROPOSED]`, cite `[SOURCE §14]`; delete Q-RPT-08. |
| AUD-009 | `13_REPORTS_AND_DASHBOARD_SPECIFICATION.md:353,356` | 8. Payment Reports (PM-04) | "Partially Paid Payments" tagged `[PROPOSED]` / "not stated in source", gated on Q-PAY-006. The *report category* is confirmed; only the allocation formula is open. | §14: "Payment … paid, partially paid, pending and unreconciled payments" | Confirmed report downgraded to optional. | Mark report `[SOURCE §14]`; keep only allocation-formula dependency open (Q-PAY-006). |
| AUD-010 | `19_EXTERNAL_INTEGRATIONS.md:100`; `21_TESTING_STRATEGY.md:287`; `24_PHASE_ROADMAP.md:55` | Q-PAY-003 usage | Q-PAY-003 is used for three **different** open items: bank status/UTR mechanism (docs/21, 24), webhook signing (docs/19), while canonical Q-PAY-003 = **reconciliation queue states/workflow/escalation** (`docs/09:395`, `docs/25:86`). | Not in source (ID-mapping defect) | One Q-ID resolves two different decisions; wrong item gated. | Use Q-PAY-001/BNK-01 for provider & status mechanism, Q-SEC-10/BNK-03 for signing; repoint docs/19:100, 21:287, 24:55; add to docs/25 correction table. |
| AUD-011 | `00_DOCUMENTATION_INDEX.md:49,107,112–135,150,154,224`; footers in `01:544`, `02:552`, `03:793`, `04:639`, `11:258`, `12:306`, `13:570`, `14:272`, `15:866`, `16:964`, `18:364`, `19:402`, `20:202`, `22:255`, `23:256`, `24:191`; `01:519` | Index + navigation | Index lists non-existent planned names (`01_PROJECT_OVERVIEW`, `02_BUSINESS_PROCESSES`, `03_FUNCTIONAL_REQUIREMENTS`, `05_DATA_MODEL`, `12_FARMER_PORTAL_REQUIREMENTS`, `11_PHASE_2_ROADMAP`, `14_OPEN_QUESTIONS_AND_DECISIONS`) which don't match the real 00–25 inventory; claims "Only `00` exists today" (:107); 6 references to `14_OPEN_QUESTIONS_AND_DECISIONS.md` which doesn't exist (actual register = `docs/25`). All "Next in sequence" footers point to the stale names. | Not in source (file inventory) | Navigation hub and reading order are broken; open-question decisions appear undiscoverable. | Rewrite index to real file names; fix all footers; point open-question references to `docs/25`. |

### 3.3 MEDIUM

| ID | Document | Section | Problem | Source requirement | Impact | Recommended resolution |
|---|---|---|---|---|---|---|
| AUD-012 | `04_MODULE_SPECIFICATION.md:45,173`; `06_FUNCTIONAL_REQUIREMENTS.md:863` vs `07_FARMER_PORTAL_SPECIFICATION.md:373`; `00:50`; `01:523` | Module Index / OQ-01 | Farmer Portal is asserted "Phase 1" in module/FR tables while OQ-01 records delivery phase as **still to be confirmed**. | §1 confirms only "no mobile app in Phase 1"; portal itself is `[NEW]` with phase undecided. | Engineering may commit portal to Phase 1 before approval, or planning breaks if phase changes. | Resolve OQ-01, or mark every Phase-1 portal reference "(phase pending OQ-01)". |
| AUD-013 | `24_PHASE_ROADMAP.md:153`; `18_SECURITY_AND_PRIVACY.md:136,160`; `19_EXTERNAL_INTEGRATIONS.md:57,292`; `25_OPEN_QUESTIONS_AND_ASSUMPTIONS.md:216` | Future / INT-04 / KYC | KYC captured/labelled "future-ready `[SOURCE §20]`". KYC is a **Phase-1 farmer-master field** (§3); §20's future-ready list is Area, Collection centre, GPS, Quality/grade, Lot, Batch, Warehouse, Packing, Export shipment — no KYC. Only *document storage* is future. | §3: "KYC information/documents where required" | Phase-1 KYC capture may be deferred/dropped. | Tag KYC capture `[SOURCE §3]` (conditional Phase 1); keep only storage/AV pipeline as `[PROPOSED]`/future (Q-INT-16). |
| AUD-014 | `22_DEPLOYMENT_AND_DEVOPS.md:99,251`; `23_SCALABILITY_AND_PERFORMANCE.md:82,240`; `24_PHASE_ROADMAP.md:55,187` vs `15_DATABASE_DESIGN.md:853`; `25:175,291` | Q-DB-01 usage | Docs 22/23/24 use "Q-DB-01" for **DB technology selection**, while docs/15 defines Q-DB-01 as the **outstanding/opening/closing computation rule**. Master register introduces Q-DB-11 (DB-08) but docs 22/23/24/15 are not updated. | Not in source (ID-mapping defect) | Two unrelated decisions share one Q-ID; DB-vendor gate (go-live critical) is mis-tracked. | Repoint docs/22:99,251, 23:82,240, 24:55,187 to Q-DB-11/DB-08; add Q-DB-11 row to docs/15. |
| AUD-015 | `19_EXTERNAL_INTEGRATIONS.md:106,116,345`; `21_TESTING_STRATEGY.md:89,289` vs `01:181,535`, `02:143,500,543`, `07:107,375`, `15:202,855`, `25:51` | Idempotency vs identity dedup | "OQ-13" is reused for **request-boundary idempotency (duplicate credits)** in docs/19/21, while OQ-13 is canonically **same-mobile → two Farmer IDs** identity deduplication. Two unrelated "dedup" concepts share one ID. | Not in source (ID-mapping defect) | Requirement tracing corrupted; idempotency decision orphaned (no owning register row). | Add a distinct idempotency row (Q-API/Q-INT) in docs/25; replace OQ-13 citations in docs/19/21. |
| AUD-016 | `18_SECURITY_AND_PRIVACY.md` (whole doc) | §18 coverage | The §18 bullet "Protection against common web/API attacks" has no mapping in the security spec (no injection/XSS/CSRF coverage). It exists only in `docs/06:109`, `docs/01:397`, `docs/21:184`. | §18: "Protection against common web/API attacks" | Security spec incomplete for a confirmed control; testing doc must own it. | Add an OWASP/common-attacks section mapping the control to `[SOURCE §18]`. |
| AUD-017 | `18_SECURITY_AND_PRIVACY.md:57`; `25_OPEN_QUESTIONS_AND_ASSUMPTIONS.md` (§5 Auth) | Authentication | OTP/2FA recorded only `[PROPOSED]` and the master register has **no open row for the OTP/2FA applicability scope** ("where appropriate"). | §18: "OTP/2FA where appropriate" | Privileged-account 2FA may silently default to not-implemented. | Tag OTP/2FA as `[SOURCE §18]`; add an A-xx/Q-SEC row for scope (staff vs privileged vs farmer). |
| AUD-018 | `18_SECURITY_AND_PRIVACY.md:44, §15` | Standing requirements / Session Management | Session management grouped "Mostly `[PROPOSED]`" and reduced to "login/enforced default deny". | §18: "Session management" | Confirmed session-management requirement missing from confirmed inventory. | Record session management as `[SOURCE §18]`; only token TTL/single-session rules are proposals. |
| AUD-019 | `18_SECURITY_AND_PRIVACY.md:218` | 17. Rate Limiting | Rate limiting introduced only as `[PROPOSED]`. | §18: "Rate limiting" | Rate limiting could be treated as optional hardening; contradicts docs/06 FR-AUTH-006. | Tag rate limiting as `[SOURCE §18]`; only thresholds/algorithms are proposals. |
| AUD-020 | `16_API_SPECIFICATION.md:923`; `21_TESTING_STRATEGY.md:89` | 22. Endpoint Summary | Header says "47 endpoints" but the same group table sums to **69 Phase-1 / 73 incl. Phase 2**. | Not in source (internal arithmetic) | Test/estimate coverage mis-reported. | Recount and state 69 / 73 consistently in docs/16 and docs/21. |
| AUD-021 | `17_UI_UX_SPECIFICATION.md:370` (E7) vs `16_API_SPECIFICATION.md` (Group 1) vs `18_SECURITY_AND_PRIVACY.md` (Q-SEC-04) | E7 Profile/Session | E7 action "change password (FR-AUTH-002)" has **no API endpoint** (Group 1 exposes only login/refresh/logout/OTP/me) and docs/18 marks employee self-service reset undefined. | Not in source | UI promises an unbuildable action. | Add `PATCH /auth/password` to docs/16 or mark the E7 action "pending Q-SEC-04" and route via admin. |
| AUD-022 | `13_REPORTS_AND_DASHBOARD_SPECIFICATION.md:427,439,449`; `15_DATABASE_DESIGN.md:194`; `17_UI_UX_SPECIFICATION.md:66,145` | K-KPI / Dashboard / DB status / UI chip | "Blocked farmers" KPI + widget + DB status note tagged `[SOURCE §15]`; §15 has **no blocked-farmer metric** (only Total/Active). "block/unblock farmer" UI action is un-tagged. | §15: "Total farmers, Active farmers, …" | Dashboard/UI scope creep anchored to false source citations. | Relabel blocked-farmer as `[NEW]`/derived-from-status; tag block/unblock action `[NEW]`/`[PROPOSED]`; align status values with docs/15. |
| AUD-023 | `13_REPORTS_AND_DASHBOARD_SPECIFICATION.md:435–436` | 10.2 Dashboard Visuals | "Employee-wise performance" and "Daily activity (daily payments)" tagged `[SOURCE §15]`. | §24: "Employee-wise performance" (Phase 2); §15 has no daily-payments widget | Phase-2 metrics misrepresented as Phase-1 §15-mandated widgets. | Tag employee-wise performance `[SOURCE §24]` (Phase 2); mark daily-activity widget `[NEW]`/DERIVED. |
| AUD-024 | `13_REPORTS_AND_DASHBOARD_SPECIFICATION.md:460–466` | 11. Phase 2 Area-wise Reporting | Section omits §24 employee-wise performance metrics (farmers handled, invoices created, quantity purchased, purchase value, average rate). | §24 | Phase-2 reporting scope under-specified. | Enumerate all §24 metrics in the section with `[SOURCE §24]`. |
| AUD-025 | `14_AUDIT_LOGGING_AND_DATA_INTEGRITY.md:176–179` | 6. Immutable Historical Financial Records | "treat history as append-only" sits inside the `[SOURCE §17]` core-rule block; absolute append-only is stricter than the source (doc itself tags it `[PROPOSED]` in §12). | §17: "Historical financial records must not be silently overwritten." | Append-only may be read as confirmed, over-constraining corrections/reversals. | Keep :177–178 as `[SOURCE §17]`; tag :179 append-only as `[PROPOSED]`. |
| AUD-026 | `20_NOTIFICATION_SPECIFICATION.md:63–88` | §4 Notification Lifecycle | Mandated status set "generated/sent/delivered/failed/retry" (§13) has no explicit **Generated** state in the lifecycle model (Created→Queued→Sent…) — only a prose cross-ref at :63 (the state exists in docs/11:157 and docs/10:206); `docs/21:174` retry matrix likewise omits it. | §13: "Record generated/sent/delivered/failed/retry status" | First mandated status lost from state machine and tests. | Add explicit `Generated` initial state in docs/20 §4 and docs/21 retry matrix. |
| AUD-027 | `21_TESTING_STRATEGY.md:119,222–237` | §6 SC-01/SC-02 | No test asserts the prohibition "Employees must not manually type invoice numbers"; SC-01/02 only prove auto-assignment/uniqueness. | §6: "Employees must not manually type invoice numbers." | A build exposing manual invoice-number entry could pass the suite. | Add negative test: manual invoice-number input rejected; number server-derived only. |

### 3.4 LOW

| ID | Document | Section | Problem | Source requirement | Impact | Recommended resolution |
|---|---|---|---|---|---|---|
| AUD-029 | `22_DEPLOYMENT_AND_DEVOPS.md:103` | 3.3 Database | "Backups (§17)" — wrong reference; backups are docs/18 §21/§22; §17 is rate limiting (PDF §17 is Audit). | §18 (backups) | Reader chases wrong requirement. | Change to "(docs/18 §21/§22)". |
| AUD-030 | `22_DEPLOYMENT_AND_DEVOPS.md:172,176` | 7.1/7.2 Backup, Restoration & DR | Off-site copy, integrity hashes, quarterly restore-drill cadence written **inside REQUIREMENT rows** despite being open (Q-SEC-15/16, S-08/S-09). | §18 (backups) only | Unapproved commitments presented as requirements. | Mark those specifics `[PROPOSED]` and reference Q-SEC-15/16. |
| AUD-031 | `23_SCALABILITY_AND_PERFORMANCE.md:250` | §B Source Reaffirmation | Fabricated verbatim quote "System should be designed in such a way that it can handle higher volume in the future" attributed to `[SOURCE §19]`. | §19: "Design for transaction volume, not just farmer count." / "Avoid fundamental redesign." | Fabricated citation weakens traceability. | Rephrase without quotes citing §19/§1 substance. |
| AUD-032 | `19_EXTERNAL_INTEGRATIONS.md:374` | §9 Source Reaffirmation | "Statements are usual PDF format" cited `[SOURCE §13, §12]`; only §13 mentions PDF statement. | §13 only | Traceability pollution. | Drop §12 from the citation. |
| AUD-033 | `19_EXTERNAL_INTEGRATIONS.md:82` | 3.2 Business Workflow step 4 | Auto status/UTR receipt + auto ledger update cited `[SOURCE §9, §10]`; the automatic reconciliation mandate is §11. | §11 | Wrong antecedent. | Cite `[SOURCE §11]`. |
| AUD-034 | `20_NOTIFICATION_SPECIFICATION.md:57` | §3 Notification Channels | "(NEW no app in Phase 1)" mislabels a source statement as a new-portal requirement. | §1: "Farmers will not have a separate mobile application in Phase 1." | Convention table violated. | Tag as `[SOURCE §1]`. |
| AUD-035 | `12_RBAC_AND_AUTHORIZATION.md:33` | 2. Role Definitions | SUPER_ADMIN row cites `[SOURCE §2]` but includes "configures the permission matrix", which is §16; Admin↔Super Admin equivalence asserted implicitly. | §16 | Minor trace inaccuracy. | Tag `[SOURCE §2, §16]` and note the mapping. |
| AUD-036 | `12_RBAC_AND_AUTHORIZATION.md:100–101` | 5. Action-Level Permissions | "Update farmer = No / View farmer = Limited" tagged literal `[SOURCE §16]`; §16 grants procurement-read access only and does not enumerate CRUD | §16 | Literal-reading of a derived permission. | Tag as derived-from-§16 / `[PROPOSED]` baseline. |
| AUD-037 | `09_PAYMENT_AND_RECONCILIATION_SPECIFICATION.md:55,61,70,74` | §5 Payment Lifecycle | Diagram/caption claim "Created" state and "manual confirmation without bank step" are `[SOURCE §11]`-mandated; §11 lists exactly five statuses (matched/unmatched/failed/pending/duplicate) and the doc itself marks manual fallback undefined (09:225, Q-PAY-005). | §11 | Manual payment confirmation may be treated as confirmed. | Tag "Created" + manual transition as added/undefined. |
| AUD-038 | `09_PAYMENT_AND_RECONCILIATION_SPECIFICATION.md:261,287,336` | 28. Exception Handling | Queue membership "unmatched, failed, pending, duplicate" tagged `[SOURCE §11]`; §11 mandates the statuses and the queue's existence, not the membership (Q-PAY-003). | §11 | Inference presented as source-attributed. | Tag membership as inferred pending Q-PAY-003. |
| AUD-039 | `08_PROCUREMENT_AND_INVOICE_SPECIFICATION.md:298` vs `:314` | 27. Business Rules – Consolidated | Header "All rules below are **source rules** … No rule is invented" contradicts PINV-BR-13 tagged `[NEW]`. | Internal wording contradiction | Trust/consistency issue for reviewers. | Reword header to "source rules or explicitly tagged `[NEW]`". |
| AUD-040 | `08_PROCUREMENT_AND_INVOICE_SPECIFICATION.md:296–314` | 27. Business Rules – Consolidated | Consolidated PINV-BR list omits the WhatsApp purchase notification rule (§8) though covered at 08:93/283. | §8 | Consumers may miss a procurement-lifecycle rule. | Add PINV-BR-14 "auto WhatsApp purchase notification `[SOURCE §8]`". |
| AUD-041 | `07_FARMER_PORTAL_SPECIFICATION.md:102` | §6 Farmer Identity Mapping | "identity anchor for WhatsApp and (portal) login seed" attributed wholly to `[SOURCE §3, §21]`; the portal-login seed is a `[NEW]` concept. | §3/§21 (WhatsApp identity only) | Portal-login seeding may be read as PDF-confirmed. | Split cell: login seed `[NEW]`/OQ-02; keep `[SOURCE §3, §21]` for WhatsApp anchor. |
| AUD-042 | `07_FARMER_PORTAL_SPECIFICATION.md:233` | §12 Screen: My Invoices | "status" field inside the `[SOURCE §7]` confirmation-attribution; §7 confirmation has no status field. | §7 example fields | Minor misattribution. | Move "status" out of the `[SOURCE §7]` tag or mark `[NEW]`. |
| AUD-043 | `06_FUNCTIONAL_REQUIREMENTS.md:683–695,1143–1155,1005–1018` | FR-LED/FR-AUD/FR-PRT | "No silent overwrite" stated in both FR-LED-003 and FR-AUD-003; own-data-only stated in FR-PRT-012 and again in docs/07 §7. | §17; `[NEW]` isolation | Duplicated rules risk drift; no owner. | Keep one owner per rule; the other references it. |
| AUD-044 | `05_USER_ROLES_AND_USER_STORIES.md:352` | US-033 | Purchase-notification story cites `[SOURCE §5]`; the automated WhatsApp rule is §8. | §8 | Traceability noise. | Cite `[SOURCE §8]`. |
| AUD-045 | `06_FUNCTIONAL_REQUIREMENTS.md:581` | FR-REC-001 | "The system MUST follow: Portal → Bank/API → Payment → UTR/status → Portal" upgrades the source's "Desired workflow" phrasing to unqualified MUST (FR-REC-002 carries the "wherever supported" caveat). | §11: "Desired workflow: …" | Integration may be treated as guaranteed rather than provider-dependent. | Mirror the provider-capability qualifier in FR-REC-001. |
| AUD-046 | `13_REPORTS_AND_DASHBOARD_SPECIFICATION.md:84,200` | 6. Farmer Reports (F-03) | F-03 Farmer Outstanding cites `[SOURCE §15]` (aggregate KPI); the farmer-wise outstanding *report* is the §14 category. | §14 Farmer row: "…purchase, payment, outstanding, ledger, monthly statement" | Report-to-source trace slightly wrong. | Cite `[SOURCE §14]` for the report. |
| AUD-047 | `13_REPORTS_AND_DASHBOARD_SPECIFICATION.md:95,97,98,344,368,377` | 8. Payment Reports (PM-03/05/06) | Rows cite only `[SOURCE §11]` statuses; "paid, pending, unreconciled" are §14 categories; "Paid" is not a §11 status word. | §14 Payment row; §11 status list | Partial trace; "Paid" pseudo-status drift risk. | Add `[SOURCE §14]`; map "Paid" explicitly to §14 category / Matched state. |
| AUD-048 | `13_REPORTS_AND_DASHBOARD_SPECIFICATION.md:32,62–76` | 3. Report Centre | R-02 "Invoice Register" and R-06 "Monthly Summary" are not §14 category names (derived from docs/01–02 registers); R-09 is §24. | §14 categories only | Overstated §14 mandate attribution. | Annotate origins (docs/01–02) and R-09 as `[SOURCE §24]`. |
| AUD-049 | `14_AUDIT_LOGGING_AND_DATA_INTEGRITY.md:43,185,191` | 7. Corrections | "Corrections are made by a new entry or reversal" presented under `[SOURCE §17]`; §17 only forbids silent overwrite. | §17 | Design mechanism may be treated as mandated. | Tag "new entry or reversal" as `[PROPOSED]`; keep "no silent edit" as `[SOURCE §17]`. |
| AUD-050 | `14_AUDIT_LOGGING_AND_DATA_INTEGRITY.md:52` | 3. Integrity Sources | "Reversal/mismatch workflow on reconciliation" cites `[SOURCE §10]` (payment fields) — mismatch/reconciliation is §11. | §11 | Wrong antecedent. | Re-cite `[SOURCE §11]`. |
| AUD-051 | `16_API_SPECIFICATION.md:597` | 12.4 GET /invoices/{invoiceNumber}/pdf | "FARMER own via portal" role listed, but Farmer Portal group has no invoice-PDF endpoint and OQ-07 keeps it open. | Not in source (OQ-07 open) | Role matrix promises a capability with no endpoint. | Add group-5 invoice-PDF endpoint or drop the FARMER role pending OQ-07. |
| AUD-052 | `15_DATABASE_DESIGN.md:60` | E19 | "See §18" points to non-existent section (future-ready entities are §5). | Not in source | Broken cross-reference. | Change to "See §5 (Future-Ready Fields)". |
| AUD-053 | `15_DATABASE_DESIGN.md:592–609` | 6. ER Diagram | FARMER block omits `products_normally_supplied` (§3 field present at E05 :195). | §3: "Products normally supplied" | Implementer coding from ER may drop a mandated field. | Add the field to the ER FARMER block (verify no other §3 fields dropped). |
| AUD-054 | `17_UI_UX_SPECIFICATION.md:66` | 3. Design System — Status chip | "Active/Blocked" chip attributed to `[SOURCE §11, §13]`; neither defines those values (they define payment/statement statuses). | §11/§13 = payment & statement statuses | Wrong provenance for a UI state. | Cite open status OQ or tag `[NEW]`/`[PROPOSED]`. |
| AUD-055 | `18_SECURITY_AND_PRIVACY.md:55,90,359` | 3/6/28 Authentication | "No default passwords" asserted as `[SOURCE §4]` / `[SOURCE §18, §4]`; neither states "default" or "no default". | §4 "Authentication credentials"; §18 "Strong password policy" | Unstated rule presented as stated. | Derive explicitly from "Strong password policy" or re-tag `[PROPOSED]`. |
| AUD-056 | `19_EXTERNAL_INTEGRATIONS.md:93` | 3.3 Data Exchanged | "Farmer-bank data minimisation pending Q-PAY-001/Q-PAY-002" — Q-PAY-002 is payment sync frequency, unrelated. | n/a (ID-mapping) | Decision references misdirect. | Alias to BNK-07/Q-INT-15 (Q-PAY-001 may stay for provider/DPA context). |
| AUD-057 | `21_TESTING_STRATEGY.md:237` | §6 SC-02 / Traceability | Dangling traceability ID "TSC-01 per docs/09" — not defined in docs/09 or elsewhere. | n/a | Regression linkage unverifiable. | Define TSC-01 or reference SC-01/SC-02/Q-PAY-006. |
| AUD-058 | `23_SCALABILITY_AND_PERFORMANCE.md:132` | 10. Large Ledger Performance | "tens of thousands of entries cumulative (SC-12 target)" — invented per-farmer figure; SC-12 is ~1M rows across 50k farmers. | §19 (millions of records) | Capacity/rolling design skewed by unwarranted assumption. | Qualify as `[PROPOSED]` example or tie to a defined per-farmer profile (PERF-08). |
| AUD-059 | `24_PHASE_ROADMAP.md:101–104` | 4.2 Phase 2 Features | Phase 2 feature list does not explicitly enumerate §23 admin functions (create/manage areas, assign/reassign, transfers, area-wise performance) or §24 employee-performance metrics. | §23, §24 | Roadmap alone fails the "Phase 2 covers §21–§24" completeness test. | Name the §23 functions and §24 metrics explicitly (compact list). |
| AUD-060 | `11:29,258`; `12:306`; `13:570`; `14:272`; `15:866`; `16:964`; `18:364`; `19:402`; `20:202`; `22:255`; `23:256`; `24:191` | Document footers / internal refs | "Next in sequence" footers name non-existent successors (stale naming scheme); `docs/11:29` references a non-existent "§70". | Not in source | Readers follow the sequence onto wrong files. | Correct footers to the real 00–25 inventory; fix the "§70" ref. |

### 3.5 INFO

| ID | Document | Section | Problem | Source requirement | Impact | Recommended resolution |
|---|---|---|---|---|---|---|
| AUD-061 | `06_FUNCTIONAL_REQUIREMENTS.md:1257` (and `00:90`, `01:153`, `02:432`, `03:737`) | FR-P2-05 / source-end note | The source PDF ends mid-sentence at "Corrections and cancellations". docs/06 embeds it inside the §24 reporting row tagged `[SOURCE §24]`; other docs treat it as a Phase-2 reporting scope phrase — inconsistent handling of truncated source content. | Source footnote (truncated); §24 lists reports then stops | A "corrections & cancellations report" may be built as if §24-defined when the source defines nothing. | Add an explicit "SOURCE TRUNCATED at §24-end" note; treat corrections/cancellations requirements as `[NEW]`/open (OQ-08), not `[SOURCE §24]`. |
| AUD-062 | `21_TESTING_STRATEGY.md` (§5) | Q-TST id numbering | Q-TST-09…Q-TST-18 are unused/reserved; numbering jumps from Q-TST-08 to Q-TST-19. | n/a | Cosmetic; minor trace hygiene. | Renumber or document the reservation. |
| AUD-063 | `16_API_SPECIFICATION.md:929` (group 2) | 22. Endpoint Summary | Group 2 row reads "(3+1)" — only 3 explicitly named auth/admin… verify Phase-1 group counts during the AUD-020 recount to keep the summary and catalog equal. | n/a | Recount housekeeping. | Fold into AUD-020 correction pass. |

---

## 4. Audit Checklist Coverage (26 requested checks)

| # | Requested check | Status | Findings |
|---|---|---|---|
| 1 | Requirements from PDF that are missing | Mostly covered; gaps: "Protect against common web/API attacks" unmapped in docs/18 (AUD-016); "manual invoice entry" not regression-tested (AUD-027); §24 employee metrics missing from docs/13 §11 (AUD-024). | AUD-016, 024, 027 |
| 2 | Requirements accidentally changed | §19 millions-of-records relabeled user-stated (AUD-005); §14 CSV/P-05/P-06/PM-04 relabeled proposed (AUD-007/008/009); "resets monthly" invented (AUD-006); KYC moved to future (AUD-013); "Desired workflow"→MUST (AUD-045). | AUD-005–009, 013, 045 |
| 3 | Contradictory requirements | Portal Phase-1 vs OQ-01 open (AUD-012); docs/18:360 vs docs/22/24 backups status (AUD-004); docs/23:28 vs docs/01/00/15 (AUD-005); docs/14 append-only vs §17 (AUD-025); ID collisions (AUD-010, 014, 015). | AUD-004, 005, 010, 012, 014, 015, 025 |
| 4 | Duplicate requirements | "No silent overwrite" in FR-LED-003 & FR-AUD-003; own-data rule in FR-PRT-012 & docs/07 (AUD-043). | AUD-043 |
| 5 | Missing business rules | §6 no-manual-typing not tested (AUD-027); consolidated PINV list misses §8 notification (AUD-040); statements/statuses Generated state (AUD-026). | AUD-026, 027, 040 |
| 6 | Missing user roles | None missing (Super Admin, Employee, + Farmer `[NEW]` verified). | — |
| 7 | Missing permissions | None missing; permission-matrix-derived archive rows need DERIVED tags (AUD-036); SUPER_ADMIN §2/§16 conflation (AUD-035). | AUD-035, 036 |
| 8 | Missing validation | Covered in docs/06/16 (qty×rate server-side, unique, no-manual-entry). Gap: negative test for no-manual-entry (AUD-027). | AUD-027 |
| 9 | Missing error cases | Recon queue membership inference (AUD-038); "Created" manual path (AUD-037). | AUD-037, 038 |
| 10 | Missing audit requirements | All §17 fields present (verified); append-only overclaim (AUD-025); corrections mechanism tag (AUD-049). | AUD-025, 049 |
| 11 | Missing security requirements | Transfers/backups/restore/session/ratelimit/2FA misclassified in docs/18 (AUD-001/002/003/017/018/019); no common-attacks section (AUD-016). | AUD-001/002/003, 016–019 |
| 12 | Missing payment scenarios | Full/partial/multi-per-invoice/multi-invoice-per-payment present (verified); queue-flow inference (AUD-038); status vocabulary "Paid" (AUD-047). | AUD-038, 047 |
| 13 | Missing invoice scenarios | Present (uniqueness, cancel-retained, no-reuse) — verified; negative no-manual-entry test (AUD-027); "resets monthly" (AUD-006). | AUD-006, 027 |
| 14 | Farmer data isolation | Present + consistent ([NEW] own-data, 404 semantics, DB-level) — verified. | — |
| 15 | Missing scalability requirements | Present; 'millions' mislabeled (AUD-005); invented per-farmer figure (AUD-058). | AUD-005, 058 |
| 16 | Missing Phase 2 requirements | Chatbot/areas/admin/reporting present (verified); §24 report metrics under-enumerated in docs/13 §11 (AUD-024) and roadmap §4.2 (AUD-059). | AUD-024, 059 |
| 17 | Farmer Login inconsistencies | Phase vs OQ-01 (AUD-012); login-seed attribution (AUD-041); change-password no API (AUD-021); no 2FA scope (AUD-017). | AUD-012, 017, 021, 041 |
| 18 | Database/API inconsistencies | Endpoint count 47 vs 69/73 (AUD-020); invoice-PDF FARMER role (AUD-051); E19 §18 ref (AUD-052); ER field omission (AUD-053); Q-DB-01/11 (AUD-014). | AUD-014, 020, 051, 052, 053 |
| 19 | UI/API inconsistencies | Change-password with no endpoint (AUD-021); status chip attribution (AUD-054); blocked-farmer UI (AUD-022). | AUD-021, 022, 054 |
| 20 | Undefined assumptions | Q-PAY-003/Q-DB-01/OQ-13 collisions (AUD-010/014/015); "no default passwords" mislabel (AUD-055); blocked status values (AUD-022). | AUD-010, 014, 015, 022, 055 |
| 21 | Confirmed presented as proposed | docs/18 transit/backups/restore/sessions/ratelimit/2FA (AUD-001/002/003/017/018/019); docs/13 CSV/P-05/P-06/PM-04 (AUD-007/008/009); docs/23 millions (AUD-005). | AUD-001/002/003, 005, 007–009, 017–019 |
| 22 | PDF requirements incorrectly labeled proposed | Same set as #21 → covered by those findings. | AUD-001/002/003, 005, 007–009 |
| 23 | Feature with no business purpose | "Blocked-farmer" KPI/widget/status (AUD-022); "Daily activity" widget (AUD-023) — no source purpose; need [NEW]/derived tags. | AUD-022, 023 |
| 24 | API without corresponding FR | Change-password (no endpoint — AUD-021); invoice-PDF FARMER role without endpoint (AUD-051); corrections endpoint absent though "Corrections and cancellations" appears in scope (AUD-061). | AUD-021, 051, 061 |
| 25 | DB entity without business requirement | None found (all entities traced to §3–§13, §16–§20, §22–§23) — verified. | — |
| 26 | UI module without FR | None found; E7 change-password is the borderline (action exists in UI, FR exists, but no API — AUD-021). | AUD-021 |

---

## 5. Verified-OK Areas (confirmed consistent, no action)

- **§1–§2:** Project objective, transaction chain *Farmer → Purchase → Invoice → Ledger → Payment → UTR → Monthly Statement → WhatsApp*, 10,000/100 → 50,000/500 figures, no-farmer-mobile-app-in-Phase-1, Super Admin/Employee role definitions — exact across docs 00–04.
- **§3–§4:** Farmer Master & Employee Master fields, permanent unique Farmer ID, mobile↔Farmer-ID WhatsApp linkage, area reserved for Phase 2.
- **§5–§8:** Procurement workflow + qty×rate auto-compute, all 12 fields; all eight §6 invoice-numbering rules; §7 confirmation example values; §8 auto WhatsApp purchase notification.
- **§9–§12:** Farmer-wise ledger columns; payment module fields + all four payment models (full, partial, multiple-per-invoice, multiple-invoices-per-payment); §11 workflow + five statuses + exception queue (with manual fallback correctly held undefined); §12 payment notification.
- **§13:** Monthly statement confirmed Phase 1, 1st-of-month/previous-month, PDF, WhatsApp, generated/sent/delivered/failed/retry statuses — consistent across docs/10, 13, 15, 16, 20, 21, 23, 24.
- **§14–§17:** Report catalogue majority, all 12 §15 dashboard metrics + date filters, §16 permission matrix configurable by Admin, §17 audit fields.
- **§18:** All 13+1 security controls enumerated in the doc set (issue is only labels in docs/18 for 6 of them — see AUD-001/002/003/017/018/019).
- **§21–§24 (Phase 2):** Nine chatbot queries, area assignment incl. "search/view/edit/download" + backend/database enforcement, Phase-2 admin functions, area/employee reporting — represented across docs/11, 12, 13, 24, 01, 02.
- **Farmer data isolation:** consistent across docs/15 (DI, TC-05), 16 (§4 `/farmer/me/…`), 17 (P-ISO), 18 (§9) — own-data only, server/DB enforced, 404 non-disclosure.
- **`[SOURCE §n]` validity:** every citation points to an existing section in §1–§24 (the only boundary issue is the truncated "Corrections and cancellations" text — AUD-061).
- **Tag hygiene overall:** `[NEW]`/`[SOURCE]`/`[PROPOSED]`/undefined usage is otherwise disciplined; no vendor/provider invented; no SMS/email/push assumed.

---

## 6. Recommended Action Plan

1. **Fix CRITICAL (blocking confidence):** re-classify §18 transit/backups/restore-testing/sessions/ratelimit/2FA as `[SOURCE §18]` in `docs/18` and correct the §28 blanket "everything else [PROPOSED]" line.
2. **Fix HIGH (tracing/accuracy):**
   - Restore §19 millions-of-records as `[SOURCE §19]`; reframe PERF-08/Q-TST-20 as dataset-profile confirmation.
   - Delete the invented "resets monthly" rule in `docs/14`.
   - Relabel CSV / P-05 / P-06 / PM-04 as `[SOURCE §14]` in `docs/13`; close Q-RPT-07/08/09 accordingly.
   - Resolve the Q-PAY-003, Q-DB-01/Q-DB-11, and OQ-13 ID collisions across docs/19, 21, 22, 23, 24, 15.
   - Reconcile `docs/00` index + all "Next in sequence" footers + `14_OPEN_QUESTIONS_AND_DECISIONS.md` references to the real 00–25 inventory.
3. **Fix MEDIUM:** resolve OQ-01 (portal phase) and OQ-02 (farmer auth) so Phase-1 module lists and open questions agree; tag KYC as `[SOURCE §3]`; add change-password API or mark pending Q-SEC-04; correct dashboard widgets and endpoint count; add Generated state and the no-manual-invoice-number negative test.
4. **Decide and document:** the truncated-source "Corrections and cancellations" scope (AUD-061) should be raised with the stakeholder as an explicit open question rather than carried as `[SOURCE §24]`.
5. **Re-run spot icons:** after edits, re-run the audit pass for the corrected rows to confirm no regressions.

---

*End of audit. Read-only — no existing documentation was modified. Findings reference exact `file:line` evidence for repair.*