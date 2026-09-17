# Agri Procurement & Farmer Management System — Documentation Index

> **Status:** Draft v0.1 — 2026-09-16
> **Location:** `docs/00_DOCUMENTATION_INDEX.md` — entry point for the entire documentation set.

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
- Detailed requirements are defined in `12_FARMER_PORTAL_REQUIREMENTS.md`.
- Exact feature list and phase placement are still to be confirmed — see OQ-01 / OQ-02 in §15.

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

From `[SOURCE §21–§24]`:

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
- **Phase 2 roadmap** (chatbot + area-wise access) is pre-defined (`[SOURCE §21–§24]`).
- `[NEW]` Farmer web portal can be extended in future phases without a mobile-app dependency.

## 10. Documentation Index

The planned documentation set. Only `00` exists today; the remaining documents are created in subsequent steps.

| # | File name | Purpose | Contains | Depends on |
|---|---|---|---|---|
| 00 | `00_DOCUMENTATION_INDEX.md` | Entry point; navigation, scope and source rules | This index: project summary, transaction chain, user types, scopes, doc index, reading order, source-of-truth rules, requirement inventory, open questions | None |
| 01 | `01_PROJECT_OVERVIEW.md` | High-level project context | Objectives, purpose, scope summaries (Phase 1/2), scale targets, future-ready capabilities | 00 |
| 02 | `02_BUSINESS_PROCESSES.md` | End-to-end business workflows | Procurement workflow, payment & reconciliation workflow, monthly statement workflow, WhatsApp notification flows, cancellation & audit handling | 01 |
| 03 | `03_FUNCTIONAL_REQUIREMENTS.md` | Requirement catalogue | Phase 1 & 2 functional requirements with IDs and source tags (`[SOURCE §n]`, `[NEW]`, `[PENDING]`) and acceptance criteria | 01, 02 |
| 04 | `04_USER_ROLES_AND_PERMISSIONS.md` | Identity & access control | User types (Super Admin, Employee, Farmer `[NEW]`), permission matrix, configurable role permissions, area-based restrictions (Phase 2) | 01, 03 |
| 05 | `05_DATA_MODEL.md` | Data architecture | Entity list; field specs (Farmer/Employee Master, Invoice, Ledger, Payment, Statement log, Notification log); invoice numbering & sequencing rules; scalability/volume notes | 03 |
| 06 | `06_INTEGRATIONS.md` | External integrations | Banking/API integration patterns, payment status/UTR handling, exception/reconciliation queue, WhatsApp integration, message templates | 03, 05 |
| 07 | `07_UI_WORKFLOW_SPECS.md` | Screen-level UX specifications | Screens & user flows: employee procurement, invoice confirmation, admin dashboard, reports, farmer portal screens `[NEW]`, responsive behaviour | 03, 04 |
| 08 | `08_API_SPECIFICATION.md` | API contracts | Internal/external API contracts, request/response, authentication, webhooks, error handling for Banking and WhatsApp providers | 03, 05, 06 |
| 09 | `09_NON_FUNCTIONAL_AND_SECURITY.md` | Non-functional & security requirements | Security controls (`§18`), audit trail (`§17`), scalability (`§19`), performance, backup/restoration, session & rate limiting, compliance | 01, 03 |
| 10 | `10_REPORTING_SPECIFICATIONS.md` | Reports catalogue & formats | Report definitions (farmer/procurement/payment), filters, export formats (Excel/PDF/CSV), scheduling & delivery | 03, 05 |
| 11 | `11_PHASE_2_ROADMAP.md` | Phase 2 roadmap | Chatbot query specs, area-wise allocation & authorization rules, Phase 2 admin functions & reporting | 03, 09 |
| 12 | `12_FARMER_PORTAL_REQUIREMENTS.md` | `[NEW]` Farmer Login & Portal | Farmer login, portal feature list, responsive web/mobile-browser scope, exclusion of mobile app in Phase 1, authentication & onboarding approach | 03, 04 |
| 13 | `13_GLOSSARY.md` | Terminology reference | Definitions: Farmer ID, Ledger, UTR, Invoice sequence, Outstanding, Statement, Area, Collection centre, etc. | 01 |
| 14 | `14_OPEN_QUESTIONS_AND_DECISIONS.md` | Decision and open-question log | Undefined items (tagged by origin), decisions and rationale, change history | All |

## 11. Recommended Reading Order (dependency-first)

1. `00_DOCUMENTATION_INDEX.md` (this file — start here)
2. `01_PROJECT_OVERVIEW.md`
3. `02_BUSINESS_PROCESSES.md`
4. `03_FUNCTIONAL_REQUIREMENTS.md`
5. `04_USER_ROLES_AND_PERMISSIONS.md`
6. `12_FARMER_PORTAL_REQUIREMENTS.md` `[NEW]`
7. `05_DATA_MODEL.md`
8. `06_INTEGRATIONS.md`
9. `07_UI_WORKFLOW_SPECS.md`
10. `08_API_SPECIFICATION.md`
11. `09_NON_FUNCTIONAL_AND_SECURITY.md`
12. `10_REPORTING_SPECIFICATIONS.md`
13. `13_GLOSSARY.md` (reference only — use at any point)
14. `11_PHASE_2_ROADMAP.md`
15. `14_OPEN_QUESTIONS_AND_DECISIONS.md` (maintained continuously alongside the above)

## 12. Source of Truth & Documentation Rules

- **Primary source of truth:** `AgriProcurement & Farmer Management.pdf` (repository root). Requirements from it are referenced by section number, e.g. `[SOURCE §6]`.
- **No silent invention:** Do not silently invent requirements.
- **No removal or alteration:** Do not remove or change existing business rules from the source document.
- **Not specified:** Anything not found in the source document must be explicitly marked **"Not specified in the source requirements."** and tracked in `14_OPEN_QUESTIONS_AND_DECISIONS.md`.
- **Enhancements:** Any suggested addition must be marked **"Proposed Enhancement."**
- **New requirements:** Approved additions (e.g., Farmer Login / Farmer Portal) are marked `[NEW]` and listed in §14.
- **Phase 1 delivery rule:** Farmers do not receive a separate native mobile application in Phase 1; the Farmer Portal is responsive web/mobile-browser based. Any later mobile app requires explicit approval.
- **Change control:** Any deviation from a source business rule must be recorded as a decision in `14_OPEN_QUESTIONS_AND_DECISIONS.md` before implementation.
- **Process rule (current step):** Producing product and technical documentation only; no application code is written in this preparation phase.

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

## 15. Undefined / Open Questions

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

*(All items above are tracked formally in `14_OPEN_QUESTIONS_AND_DECISIONS.md` as decisions are made.)*

---

**Next step:** Create `01_PROJECT_OVERVIEW.md`.