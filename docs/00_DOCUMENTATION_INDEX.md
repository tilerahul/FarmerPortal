# Agri Procurement & Farmer Management System — Documentation Index

> **Status:** v1.1 — reconciled with the delivered documentation set (docs 01–25 + audit report). 2026-09-17.
> **Location:** `docs/00_DOCUMENTATION_INDEX.md` — entry point for the entire documentation set.
> **Note on naming:** The original plan (this file's earlier draft) listed placeholder names such as `01_PROJECT_OVERVIEW.md`, `02_BUSINESS_PROCESSES.md`, `03_FUNCTIONAL_REQUIREMENTS.md`, `05_DATA_MODEL.md`, `09_NON_FUNCTIONAL_AND_SECURITY.md`, `11_PHASE_2_ROADMAP.md`, `12_FARMER_PORTAL_REQUIREMENTS.md`, `13_GLOSSARY.md` and `14_OPEN_QUESTIONS_AND_DECISIONS.md`. Those slots were delivered under their actual names listed in §10 (e.g., `01_PRODUCT_REQUIREMENTS_DOCUMENT.md`, `25_OPEN_QUESTIONS_AND_ASSUMPTIONS.md`). A glossary was not delivered as a separate document; terminology is defined in place within `04_MODULE_SPECIFICATION.md` and the PRD/BRD.

---

## 1. Project Name

**Agri Procurement & Farmer Management System**

## 2. Project Purpose

Build a central, web-based procurement and farmer management portal for an agricultural procurement and export business. Employees use the portal to purchase agricultural products directly from farmers while maintaining complete farmer records, invoices, ledgers, payments and statements. The system automates payment reconciliation (via banking/API integration), farmer communication (via WhatsApp) and monthly statement generation.

## 3. System Overview (Short)

A single web portal serves three audiences:

- **Super Admin** — full system administration, master data, settings, integrations and audit.
- **Employee** — field procurement entry against verified farmer masters, generating invoices and initiating the payment lifecycle.
- **Farmer (NEW)** — a responsive web portal where farmers log in and view their own financial information (ledger, payments, statements, purchases).

Data and process flow are built around a permanent **Farmer ID** throughout the transaction chain, and every transaction is attributed to the employee who created it.

## 4. Primary Business Transaction Chain

```
Farmer → Purchase → Invoice → Ledger → Payment → UTR → Monthly Statement → WhatsApp
```

Source: Project Objective (`[SOURCE §1]`).

## 5. Existing User Types (from Source PDF)

| User type | Source | Access summary |
|---|---|---|
| Super Admin | `[SOURCE §2]` | Full access to entire system; manage employees/farmers; view all areas, procurement, invoices, payments, ledgers, reports; manage settings, WhatsApp and banking integrations; view audit logs. |
| Employee | `[SOURCE §2]` | Individual logins; use portal while collecting material from farmers; every transaction records the creating employee. Initial target 100; architecture supports 500+. |
| Farmer (Phase 2, indirect) | `[SOURCE §21]` | Not a login user in the source document. Interacts via WhatsApp chatbot using the registered mobile number (Phase 2). |
| Farmer Portal user | `[NEW]` | New login user introduced by decision — see §6 below. Not part of the original PDF. |

## 6. Newly Added Farmer Login / Farmer Portal Requirement (NEW)

- **Category:** `[NEW]` — approved addition, not part of the original PDF.
- **Scope:** Provide a **Farmer Login** and a **Farmer Web Portal** so farmers can access their own information electronically.
- **Delivery format:** Responsive web/mobile-browser based in Phase 1. There is **no separate native mobile application** in Phase 1 unless explicitly approved later.
- The registered mobile number remains the identity anchor tied to the Farmer ID (consistent with the source WhatsApp design; `[SOURCE §3, §21]`).
- Detailed requirements are defined in `07_FARMER_PORTAL_SPECIFICATION.md`.
- Exact feature list and phase placement are still to be confirmed — see OQ-01 / OQ-02 in §15 (consolidated in `25_OPEN_QUESTIONS_AND_ASSUMPTIONS.md`).

## 7. Phase 1 Scope — Core Procurement, Payment & Statement System

From `[SOURCE §2–§20]` plus the `[NEW]` Farmer Portal:

- User types: Super Admin; Employee (`§2`)
- Farmer Master (`§3`)
- Employee Master (`§4`)
- Procurement Entry (`§5`)
- Invoice Numbering (`§6`)
- Invoice / Transaction Confirmation (`§7`)
- WhatsApp Purchase Notification (`§8`)
- Farmer Ledger (`§9`)
- Payment Module (`§10`)
- Banking / API Integration & reconciliation queue (`§11`)
- Automatic WhatsApp Payment Notification (`§12`)
- Automatic Monthly Farmer Statement (`§13`)
- Reports (`§14`)
- Admin Dashboard (`§15`)
- User Permissions (`§16`)
- Audit Trail (`§17`)
- Security (`§18`)
- Scalability (`§19`)
- Future-Ready Fields (kept inactive but architected) (`§20`)
- `[NEW]` Farmer Login + Farmer Web Portal (responsive web/browser-based)

### Phase 1 boundaries

- Farmers do **not** get a native mobile application in Phase 1 (`[SOURCE §1]`, reaffirmed).
- Area/location, collection centre, GPS, quality/grade, lot/batch, warehouse, packing, export shipment fields remain **inactive placeholders** in Phase 1 (`[SOURCE §20]`).
- WhatsApp Chatbot and area-wise access are Phase 2 (`[SOURCE §21–§24]`).

## 8. Phase 2 Scope — WhatsApp Chatbot & Area-wise Access

From `[SOURCE §21–§24]` (detailed in `24_PHASE_ROADMAP.md`):

- **WhatsApp Chatbot:** farmers interact via WhatsApp; registered mobile → Farmer ID (`[SOURCE §21]`). Queries: Outstanding, Ledger, Purchases, Payments, Last Payment, Last Purchase, Invoice Details, Statement, Payment Status.
- **Area-wise Farmer Allocation:** farmers and employees assigned to areas; employees restricted to their own area at backend/database authorization level (`[SOURCE §22]`).
- **Phase 2 Admin Functions:** area management, farmer/employee assignment, reassignment, transfers, area-wise performance (`[SOURCE §23]`).
- **Phase 2 Reporting:** area-wise farmer count / procurement / payment / outstanding; employee-wise performance; corrections and cancellations scope (`[SOURCE §24]`).

### Phase 2 boundaries

- Phase 2 content is listed for planning; detailed design happens at Phase 2 kick-off.
- Any Farmer Portal items deferred to Phase 2 are pending decision OQ-01.

## 9. Future-Ready Capabilities

- **Scale:** initial 10,000 farmers / 100 employees → target 50,000 farmers / 500 employees without fundamental redevelopment (`[SOURCE §1, §19]`).
- **Volume:** database must support potentially millions of transaction records; design for transaction volume, not just farmer count (`[SOURCE §19]`).
- **Future-ready fields** (inactive in Phase 1, architected for later) (`[SOURCE §20]`): Area, Collection centre, GPS, Quality/grade, Lot number, Batch number, Warehouse, Packing, Export shipment.
- **Phase 2 roadmap** (chatbot + area-wise access) is pre-defined (`[SOURCE §21–§24]`) and detailed in `24_PHASE_ROADMAP.md`.
- `[NEW]` Farmer web portal can be extended in future phases without a mobile-app dependency.

## 10. Documentation Index

The full delivered documentation set. Every `docs/NN_*.md` is a numbered core document (00 is this index, 01–25 are the specification set); `DOCUMENTATION_AUDIT_REPORT.md` is a companion review document (not part of the numbered sequence).

| # | File name | Purpose | Contains | Depends on |
|---|---|---|---|---|
| 00 | `00_DOCUMENTATION_INDEX.md` | Entry point; navigation, scope and source rules | This index: project summary, transaction chain, user types, Phase 1/2 scopes, doc index, reading order, source-of-truth rules, PDF-derived requirement inventory, NEW list, open-question summary | None |
| 01 | `01_PRODUCT_REQUIREMENTS_DOCUMENT.md` | Product Requirements Document (PRD) | Complete Phase 1 (and Phase 2 outline) product requirements with IDs, tags (`[SOURCE §n]`/`[NEW]`/MUST/SHOULD/MAY), acceptance criteria, §28 open questions | 00 |
| 02 | `02_BUSINESS_REQUIREMENTS_DOCUMENT.md` | Business Requirements Document (BRD) | Business rules, roles, scenarios, business constraints and assumptions, §25 open questions | 01 |
| 03 | `03_BUSINESS_WORKFLOWS.md` | End-to-end business workflows | Procurement, payment & reconciliation, monthly statement, WhatsApp notification, correction/cancellation and audit workflows | 01, 02 |
| 04 | `04_MODULE_SPECIFICATION.md` | Module Specification | Every system module: purpose, actors, responsibilities, features, inputs/outputs, business rules, dependencies, permissions, data, notifications, audit and phase | 01, 02, 03 |
| 05 | `05_USER_ROLES_AND_USER_STORIES.md` | User roles, permissions & user stories | User types (Super Admin, Employee, Farmer `[NEW]`), role permissions, permission matrix, user stories and persona constraints | 01, 02 |
| 06 | `06_FUNCTIONAL_REQUIREMENTS.md` | Functional Requirements catalogue | Cross-module FR-… requirement IDs with business rules, MUST/SHOULD/MAY priority, verification and acceptance criteria | 01, 02, 04 |
| 07 | `07_FARMER_PORTAL_SPECIFICATION.md` | `[NEW]` Farmer Portal Specification | Farmer login & web portal: feature list, screens, authentication approach, Phase 1 responsive web scope, exclusion of native mobile app | 01, 05 |
| 08 | `08_PROCUREMENT_AND_INVOICE_SPECIFICATION.md` | Procurement & Invoice Specification | Procurement entry workflow, invoice numbering & per-farmer sequence, confirmation display, cancellation rules, SC-… rules | 01, 04, 06 |
| 09 | `09_PAYMENT_AND_RECONCILIATION_SPECIFICATION.md` | Payment & Reconciliation Specification | Payment module (full/partial/multiple payments, multiple invoices per payment), allocation rules, reconciliation/exception queue, statuses (matched/unmatched/failed/pending/duplicate), PM-… rules | 08, 10, 16 |
| 10 | `10_LEDGER_AND_STATEMENT_SPECIFICATION.md` | Ledger & Statement Specification | Farmer-wise ledger, automatic monthly statement (1st of month, prior month, PDF, WhatsApp), statement statuses & retry, LS-… rules | 01, 08 |
| 11 | `11_WHATSAPP_INTEGRATION_SPECIFICATION.md` | WhatsApp Integration Specification | WhatsApp purchase/payment notifications, chatbot queries, message templates, utility vs template constraints, delivery statuses | 19, 20 |
| 12 | `12_RBAC_AND_AUTHORIZATION.md` | RBAC & Authorization | Role permission matrix, module-level access, area-wise farmer/employee allocation and backend/database authorization (Phase 2) | 05 |
| 13 | `13_REPORTS_AND_DASHBOARD_SPECIFICATION.md` | Reports & Dashboard Specification | Reports catalogue (farmer/procurement/payment/area-wise), Excel/PDF/CSV exports, admin dashboard KPIs & date filters | 06, 08, 09 |
| 14 | `14_AUDIT_LOGGING_AND_DATA_INTEGRITY.md` | Audit Logging & Data Integrity | Audit trail (employee ID, date/time, action, before/after, record, IP/device), no silent overwrite of financials, immutability and integrity rules | 01, 08, 09, 10 |
| 15 | `15_DATABASE_DESIGN.md` | Database Design | Entity/table design, fields, keys & indexes, invoice sequence storage, ledger/payment/statement tables, transaction-volume notes | 06, 08, 09, 10 |
| 16 | `16_API_SPECIFICATION.md` | API Specification | Internal/external API contracts, request/response schemas, authentication & rate limiting, webhooks, error handling | 09, 12, 15, 19 |
| 17 | `17_UI_UX_SPECIFICATION.md` | UI/UX Specification | Screen specifications, user flows, responsive/mobile-browser behaviour, component & accessibility guidance | 05, 07, 12 |
| 18 | `18_SECURITY_AND_PRIVACY.md` | Security & Privacy | `§18` controls: authentication, password policy, OTP/2FA, encryption (at rest / in transit), RBAC, automated backups & restoration testing, audit logging, session & rate limiting | 12, 14, 15, 16 |
| 19 | `19_EXTERNAL_INTEGRATIONS.md` | External Integrations | Banking/API integration and WhatsApp provider patterns, event flows, exception queue, provider-selection considerations | 09, 11 |
| 20 | `20_NOTIFICATION_SPECIFICATION.md` | Notification Specification | Notification matrix, message templates, delivery statuses & retry policy, channel rules (WhatsApp / in-portal) | 10, 11, 19 |
| 21 | `21_TESTING_STRATEGY.md` | Testing Strategy | Test strategy, levels & environments, test data, acceptance against requirements, regression and security testing | 06, 16, 17 |
| 22 | `22_DEPLOYMENT_AND_DEVOPS.md` | Deployment & DevOps | Environments, CI/CD, release process, backup & restore operations, monitoring, rollback | 15, 18 |
| 23 | `23_SCALABILITY_AND_PERFORMANCE.md` | Scalability & Performance | Scale targets (10k → 50k farmers / 100 → 500 employees), millions of transaction records, concurrency, performance budgets, caching & tuning | 01, 06, 15, 16 |
| 24 | `24_PHASE_ROADMAP.md` | Phase Roadmap | Phase 1 vs Phase 2 scope split, delivery roadmap, future-ready fields kept inactive (`§20`), dependencies and sequencing | 01 |
| 25 | `25_OPEN_QUESTIONS_AND_ASSUMPTIONS.md` | Open Questions & Assumptions — Master List | Canonical decision register consolidating every open question / OQ-… / Q-…-id / assumption from docs 01–24 into rows with aliases, owners, options, current assumptions and impact | All (01–24) |

**Companion documents**

| Ref | File name | Purpose |
|---|---|---|
| — | `DOCUMENTATION_AUDIT_REPORT.md` | Severity-graded cross-document consistency audit of the PDF ground truth vs docs 00–25 (AUD-…), 26-point checklist coverage, remediable findings and action plan |

## 11. Recommended Reading Order (dependency-first)

1. `00_DOCUMENTATION_INDEX.md` (this file — start here)
2. `01_PRODUCT_REQUIREMENTS_DOCUMENT.md`
3. `02_BUSINESS_REQUIREMENTS_DOCUMENT.md`
4. `03_BUSINESS_WORKFLOWS.md`
5. `04_MODULE_SPECIFICATION.md`
6. `05_USER_ROLES_AND_USER_STORIES.md`
7. `06_FUNCTIONAL_REQUIREMENTS.md`
8. `07_FARMER_PORTAL_SPECIFICATION.md` `[NEW]`
9. `08_PROCUREMENT_AND_INVOICE_SPECIFICATION.md`
10. `10_LEDGER_AND_STATEMENT_SPECIFICATION.md`
11. `09_PAYMENT_AND_RECONCILIATION_SPECIFICATION.md`
12. `14_AUDIT_LOGGING_AND_DATA_INTEGRITY.md`
13. `15_DATABASE_DESIGN.md`
14. `16_API_SPECIFICATION.md`
15. `12_RBAC_AND_AUTHORIZATION.md`
16. `18_SECURITY_AND_PRIVACY.md`
17. `19_EXTERNAL_INTEGRATIONS.md`
18. `11_WHATSAPP_INTEGRATION_SPECIFICATION.md`
19. `20_NOTIFICATION_SPECIFICATION.md`
20. `13_REPORTS_AND_DASHBOARD_SPECIFICATION.md`
21. `17_UI_UX_SPECIFICATION.md`
22. `21_TESTING_STRATEGY.md`
23. `22_DEPLOYMENT_AND_DEVOPS.md`
24. `23_SCALABILITY_AND_PERFORMANCE.md`
25. `24_PHASE_ROADMAP.md` (Phase 2 planning)

Reference / maintained continuously alongside the above:
- `25_OPEN_QUESTIONS_AND_ASSUMPTIONS.md` — read per-doc open items against this master register as each document is reviewed.
- `DOCUMENTATION_AUDIT_REPORT.md` — review companion; findings are addressed before implementation.

## 12. Source of Truth & Documentation Rules

- **Primary source of truth:** `AgriProcurement & Farmer Management.pdf` (repository root). Requirements from it are referenced by section number, e.g. `[SOURCE §6]`.
- **No silent invention:** Do not silently invent requirements.
- **No removal or alteration:** Do not remove or change existing business rules from the source document.
- **Not specified:** Anything not found in the source document must be explicitly marked **"Not specified in the source requirements."** and tracked in `25_OPEN_QUESTIONS_AND_ASSUMPTIONS.md`.
- **Enhancements:** Any suggested addition must be marked **"Proposed Enhancement."**
- **New requirements:** Approved additions (e.g., Farmer Login / Farmer Portal) are marked `[NEW]` and listed in §14.
- **Phase 1 delivery rule:** Farmers do not receive a separate native mobile application in Phase 1; the Farmer Portal is responsive web/mobile-browser based. Any later mobile app requires explicit approval.
- **Change control:** Any deviation from a source business rule must be recorded as a decision in `25_OPEN_QUESTIONS_AND_ASSUMPTIONS.md` before implementation.
- **Consolidation:** Each document keeps its own local open-question IDs (OQ-…, Q-… series); `25_OPEN_QUESTIONS_AND_ASSUMPTIONS.md` is the single master register that consolidates them (aliases marked `≡`).
- **Process rule (current phase):** Producing product and technical documentation only; no application code is written in this preparation phase.

## 13. Requirements That Came From the Original PDF

All items below originate from `AgriProcurement & Farmer Management.pdf`. Tags `[SOURCE §n]` refer to its sections.

**Project-level**
1. Web-based procurement & farmer management portal for an agricultural procurement and export business — `[SOURCE §1]`
2. Transaction chain: Farmer → Purchase → Invoice → Ledger → Payment → UTR → Monthly Statement → WhatsApp — `[SOURCE §1]`
3. Scale targets: 10,000 farmers / 100 employees (initial); 50,000 farmers / 500 employees (future) — `[SOURCE §1]`
4. Farmers have no separate mobile application in Phase 1 — `[SOURCE §1]`
5. Architecture must scale without fundamental redevelopment — `[SOURCE §19]`

**Phase 1 — Core Procurement, Payment & Statement System (`[SOURCE §2–§20]`)**
6. Super Admin login: full access; manage employees/farmers; view all areas, procurement, invoices, payments, ledgers, reports; manage settings, WhatsApp & banking integrations; view audit logs — `§2`
7. Employee login: individual logins; portal use while collecting material from farmers; transaction attribution to employee — `§2`
8. Farmer Master with permanent unique Farmer ID (e.g., F-0001) and defined fields; registered mobile linked to Farmer ID for WhatsApp — `§3`
9. Employee Master (e.g., EMP-001) with defined fields; area/location reserved for Phase 2 — `§4`
10. Procurement Entry workflow (Login → Select/Search Farmer → Select Product → Enter Weight → Enter Rate → Calculate → Confirm → Generate Invoice); amount = Quantity × Rate (auto-calculated) — `§5`
11. Invoice Numbering: auto-generated from Farmer ID + per-farmer sequence (e.g., F-0001-17); stored as separate DB fields; unique; per-farmer sequence; cancelled invoices retained; numbers not reused; modifications audited; no manual entry — `§6`
12. Invoice / Transaction Confirmation display: Farmer, Invoice, Date, Product, Quantity, Rate, Total, Employee — `§7`
13. Automatic WhatsApp purchase notification after successful invoice creation — `§8`
14. Farmer-wise Ledger maintained within the system — `§9`
15. Payment Module: Farmer ID, invoice allocation, amount, date, status, mode, bank reference, UTR, remarks; supports full/partial/multiple payments vs an invoice and multiple invoices per payment — `§10`
16. Banking/API Integration: Portal → Bank/API → Payment → UTR/status → Portal; automated status & UTR receipt; automatic ledger update; matched/unmatched/failed/pending/duplicate statuses; Exception/Reconciliation Queue for accounts staff — `§11`
17. Automatic WhatsApp payment notification on bank/API-confirmed payment — `§12`
18. Automatic Monthly Farmer Statement on the 1st of each month for the previous month; mandated fields; PDF generation; WhatsApp delivery; generated/sent/delivered/failed/retry statuses — `§13`
19. Reports catalogue (farmer / procurement / payment) with Excel, PDF, CSV export where appropriate — `§14`
20. Admin Dashboard with defined KPIs and date filters — `§15`
21. User Permissions: Admin (full) and Procurement Employee (limited); permission matrix configurable by Admin — `§16`
22. Audit Trail: employee ID, date/time, action, original value, new value, record affected, IP/device; historical financial records must not be silently overwritten — `§17`
23. Security controls: secure authentication, strong password policy, OTP/2FA where appropriate, data-at-rest & in-transit encryption, RBAC, secure API authentication, automated backups + restoration testing, audit logging, session management, rate limiting, protection against common web/API attacks; restricted access to sensitive bank/farmer data — `§18`
24. Scalability: concurrent employee usage; database support for millions of transaction records — `§19`

**Phase 2 — WhatsApp Chatbot & Area-wise Access (`[SOURCE §21–§24]`)**
25. WhatsApp Chatbot with farmer queries: Outstanding, Ledger, Purchases, Payments, Last Payment, Last Purchase, Invoice Details, Statement, Payment Status — `§21`
26. Area-wise Farmer Allocation: employees restricted to assigned areas at backend/database authorization level; no search/view/edit/download outside area — `§22`
27. Phase 2 Admin Functions: area management, farmer/employee assignment, reassignment, transfers, area-wise performance; full Admin access maintained — `§23`
28. Phase 2 Reporting: area-wise farmer count, procurement, payment, outstanding; employee-wise performance (farmers handled, invoices, quantity, value, average rate); corrections and cancellations scope — `§24`

## 14. Newly Approved Requirements (NEW)

| ID | Requirement | Status | Notes |
|---|---|---|---|
| N-01 | Farmer Login | `[NEW]` — Approved | Authentication for farmers to access a web portal. Detailed design pending (see OQ-02). |
| N-02 | Farmer Web Portal | `[NEW]` — Approved | Responsive web/mobile-browser portal for farmers to view their financial data. Detailed feature list pending (see OQ-01). |
| N-03 | No separate farmer mobile application in Phase 1 | Approved boundary | Reaffirms `[SOURCE §1]`; portal must be mobile-browser friendly. |
| N-04 | Documentation-first preparation phase | Approved process | Product & technical documentation produced before any implementation code. |
| N-05 | Requirement tagging conventions | Approved rule | `[SOURCE §n]`, `[NEW]`, "Not specified in the source requirements.", "Proposed Enhancement." |

## 15. Undefined / Open Questions (summary — master register in `25_OPEN_QUESTIONS_AND_ASSUMPTIONS.md`)

| ID | Item | Origin | Current status / notes |
|---|---|---|---|
| OQ-01 | Farmer Portal delivery phase (Phase 1 vs Phase 2) and full feature list | Not specified in the source requirements (`[NEW]`) | Which portal features (ledger, statements, payments, purchases, notifications) are in scope and when they ship. |
| OQ-02 | Farmer authentication method | Not specified in the source requirements (`[NEW]`) | OTP on registered mobile (recommended given mobile is the identity anchor) vs credentials vs both. |
| OQ-03 | Banking/API provider and integration mode | Not specified in the source requirements | Provider selection drives behavior (push/pull, webhook/polling, UTR availability timing). |
| OQ-04 | WhatsApp provider & messaging policy | Not specified in the source requirements | Meta Cloud API vs third-party provider; utility vs template message constraints for business-initiated messages. |
| OQ-05 | Product master | Not specified in the source requirements | Managed product/unit catalogue, rate entry rules, approval for rate deviations. |
| OQ-06 | Deduction rules | Not specified in the source requirements | Deduction types (flat/percentage), reasons, approvals, impact on invoice and ledger. |
| OQ-07 | Invoice output format | Not specified in the source requirements | Whether the transaction confirmation is printable/PDF invoice, and reprint rules. (Monthly statement PDF is mandated.) |
| OQ-08 | Invoice cancellation workflow depth | Partially specified in the source requirements | Retention, no sequence reuse, audit are specified; cancellation authorization and reason capture are not. |
| OQ-09 | Employee permission catalogue | Partially specified in the source requirements | Two roles given; full role catalogue and configurable matrix details not specified. |
| OQ-10 | KYC specifics | Not specified in the source requirements | Accepted documents, validation level, status lifecycle. |
| OQ-11 | Audit retention & export | Not specified in the source requirements | Retention period, review frequency, export/archival format. |
| OQ-12 | Tax / statutory handling | Not specified in the source requirements | GST/taxation, government levies, commission handling. |
| OQ-13 | Farmer identity deduplication | Not specified in the source requirements | Same mobile/WhatsApp number against multiple Farmer IDs. |
| OQ-14 | Statement generation timing & retry policy details | Partially specified in the source requirements | Day-1 prior month + delivery statuses given; exact time, timezone, retry frequency not specified. |

*(Every item above is consolidated in `25_OPEN_QUESTIONS_AND_ASSUMPTIONS.md` — the master decision register — together with all per-document Q-… series open questions and current assumptions.)*

---

**Next step:** The documentation set (docs 00–25) is delivered. Recommended next actions: (1) resolve open questions via the `25_OPEN_QUESTIONS_AND_ASSUMPTIONS.md` master register, (2) remediate the findings in `DOCUMENTATION_AUDIT_REPORT.md` (CRITICAL/HIGH first), then (3) proceed to implementation.