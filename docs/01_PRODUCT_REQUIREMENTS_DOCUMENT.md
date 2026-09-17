# Agri Procurement & Farmer Management System
## Product Requirements Document (PRD)

---

## 1. Document Information

| Item | Value |
|---|---|
| Product name | Agri Procurement & Farmer Management System |
| Document | Product Requirements Document (PRD) |
| Version | v1.0 |
| Status | Draft — pending review and closure of open questions (§28) |
| Date | 2026-09-16 |
| Prepared by | Product & Documentation Team |
| Audience | Product, Engineering, QA, Solution Architecture, Business/Accounts, Leadership |
| Purpose | Define the complete, unambiguous product requirements for Phase 1 (and Phase 2 outline) so that design, build and acceptance can proceed without re-interrogating business users |
| Source | 1. `AgriProcurement & Farmer Management.pdf` (repository root) — primary source of truth 2. `docs/00_DOCUMENTATION_INDEX.md` — documentation framework, tagging rules and open-question register |
| File note | The index document planned this slot as `01_PROJECT_OVERVIEW.md`; per project decision this file (`01_PRODUCT_REQUIREMENTS_DOCUMENT.md`) is created instead and takes the `01` position. |

### Requirement tagging conventions used in this document

| Tag | Meaning |
|---|---|
| `[SOURCE §n]` | Requirement originates from section `n` of the source PDF `AgriProcurement & Farmer Management.pdf`. |
| `[NEW]` | Newly approved requirement (project decision) — not part of the original PDF. |
| MUST / SHOULD / MAY | Requirement priority: MUST = mandatory for the phase; SHOULD = expected unless a documented reason not to; MAY = optional / discretionary. |
| "Not specified in the source requirements." | A behaviour called out but not defined anywhere in the source PDF; listed in §28 open questions. |
| "Proposed Enhancement." | An added capability suggested by this document, not yet approved; requires explicit sign-off. |

---

## 2. Product Overview

The **Agri Procurement & Farmer Management System** is a central, web-based procurement and farmer management portal for an agricultural procurement and export business. Employees use the portal to purchase agricultural products directly from farmers, with a complete, auditable lifecycle from procurement transaction through invoice, ledger, payment and UTR reconciliation, to automated monthly statements delivered over WhatsApp.

The system is built around a **permanent Farmer ID** that is the primary business identity for every farmer, and an **Employee ID** that attributes every transaction to the employee who created it. Payments are reconciled through a banking/API integration, and farmers are kept informed automatically via WhatsApp notifications.

Three audiences are served by a single portal:

- **Super Admin** — full administration, master data, settings, integrations and audit.
- **Procurement Employee** — field-facing procurement entry and invoice generation.
- **Farmer (`[NEW]`)** — a responsive web portal through which farmers log in and view their own purchases, invoices, payments, ledger and statements. The Farmer Login/Portal is a newly approved requirement (see §6 and §10).

Source: `[SOURCE §1–§3]`, index §1–§6.

---

## 3. Problem Statement

A procurement and export business purchases agricultural produce from a large, distributed base of farmers. Without a centralised system, this operation suffers from:

- No single, trustworthy farmer identity — records scatter across branches, employees and paper.
- Manual, error-prone invoicing with collision-prone numbering.
- No farmer-wise financial view; farmers cannot be shown a clear picture of purchases and payments.
- Slow, opaque payment confirmation — payment status and UTRs are not centrally reconciled.
- No automated, law-consistent monthly statements, and no reliable channel to deliver them.
- Farmers have no self-service visibility of their own transactions, balances and statements (`[NEW]` — portal decision).

The effort is currently at a scale of ~10,000 farmers and ~100 employees, heading toward 50,000 farmers and 500 employees without a fundamental redesign (`[SOURCE §1, §19]`).

---

## 4. Product Objective

Build a central web-based procurement and farmer management portal that:

1. Records every farmer purchase with a permanent, unique identity and full audit (`[SOURCE §2, §3, §17]`).
2. Generates invoices automatically — numbered by Farmer ID and per-farmer sequence, never manually typed, never reused after cancellation (`[SOURCE §6]`).
3. Maintains a live farmer-wise ledger and automated payment reconciliation via banking/API integration (`[SOURCE §9–§11]`).
4. Automatically notifies farmers over WhatsApp on purchase and confirmed payment (`[SOURCE §8, §12]`).
5. Automatically produces and delivers a monthly farmer statement (PDF) on the 1st of every month (`[SOURCE §13]`).
6. Scales from 10,000 farmers / 100 employees to 50,000 farmers / 500 employees without fundamental redevelopment (`[SOURCE §1, §19]`).
7. Provides farmers a responsive web portal (`[NEW]`) for self-service access to their own financial data — no native mobile app in Phase 1.

---

## 5. Business Transaction Chain

```
Farmer → Purchase → Invoice → Ledger → Payment → UTR → Monthly Statement → WhatsApp
```

- **Farmer:** farmer is onboarded in the Farmer Master with a permanent unique Farmer ID; the registered mobile is linked to the Farmer ID.
- **Purchase:** an employee enters the procurement transaction (farmer, product, weight, rate).
- **Invoice:** a unique invoice is auto-generated from Farmer ID + transaction sequence (e.g., `F-0001-17`).
- **Ledger:** the transaction is posted to a farmer-wise ledger.
- **Payment:** payments (full, partial, or allocated across invoices) are recorded against invoices.
- **UTR:** payment status and UTR are received through the banking/API integration and the ledger is updated automatically.
- **Monthly Statement:** on the 1st of each month the previous month's statement is generated.
- **WhatsApp:** purchase confirmations, payment confirmations and monthly statements are delivered to the farmer over WhatsApp.

Source: `[SOURCE §1]`; index §4.

---

## 6. Target Users

| User type | Origin | Description / access |
|---|---|---|
| Super Admin | `[SOURCE §2]` | Full access to the entire system: manage employees and farmers; view all areas, procurement, invoices, payments, ledgers and reports; manage settings, WhatsApp and banking integrations; view audit logs. |
| Procurement Employee | `[SOURCE §2]` | Individual login per employee. Uses the portal while collecting material from farmers. Every transaction records the employee who created it (backend trail). Initial target 100 employees; architecture supports at least 500. Access limited to farmer information required for procurement and procurement entry (`[SOURCE §16]`). |
| Farmer | `[SOURCE §21]` / `[NEW]` | In the source PDF, farmers had **no login**; they interacted only via WhatsApp chatbot in Phase 2. The **Farmer Login / Farmer Portal is a newly approved requirement (`[NEW]`)** — a responsive web/mobile-browser based portal where farmers see their own data. No native mobile application in Phase 1. |

Farmer Login / Farmer Portal is explicitly identified as **newly approved**; see §6 of the index and §10 of this document.

---

## 7. User Goals

| User | Goals |
|---|---|
| Super Admin | Run the whole operation from one place: manage masters, define permissions, configure WhatsApp/banking integrations, review all financial data and reports, and hold a complete audit trail across employees, farmers and transactions. |
| Procurement Employee | Quickly find a farmer, record the purchase (product, weight, rate) and produce a correctly numbered invoice and confirmation while physically at the collection point, without touching invoice numbering. |
| Farmer | (Phase 2, source) Check outstanding, ledger, purchases, payments, last payment/purchase, invoice details, statement and payment status via WhatsApp (`[SOURCE §21]`). (Phase 1, `[NEW]`) Log in to a responsive web portal to see own profile, purchases, invoices, payments, ledger and statements (§10). |

---

## 8. Product Scope

### 8.1 Phase 1 — Core Procurement, Payment & Statement System (`[SOURCE §2–§20]`)

| Area | Source |
|---|---|
| Super Admin and Employee user types | §2 |
| Farmer Master | §3 |
| Employee Master | §4 |
| Procurement Entry | §5 |
| Invoice Numbering | §6 |
| Invoice / Transaction Confirmation | §7 |
| WhatsApp Purchase Notification | §8 |
| Farmer Ledger | §9 |
| Payment Module | §10 |
| Banking / API Integration + Exception/Reconciliation Queue | §11 |
| Automatic WhatsApp Payment Notification | §12 |
| Automatic Monthly Farmer Statement | §13 |
| Reports | §14 |
| Admin Dashboard | §15 |
| User Permissions | §16 |
| Audit Trail | §17 |
| Security | §18 |
| Scalability | §19 |
| Future-Ready Fields (as inactive placeholders) | §20 |
| `[NEW]` Farmer Login + Farmer Web Portal (responsive web/browser) | New decision |

**Phase 1 boundaries.** Farmers do not receive a native mobile application in Phase 1 (`[SOURCE §1]`). Area/location, collection centre, GPS, quality/grade, lot/batch, warehouse, packing and export shipment fields remain inactive placeholders (`[SOURCE §20]`). WhatsApp Chatbot and area-wise access are Phase 2 (`[SOURCE §21–§24]`).

### 8.2 Phase 2 — WhatsApp Chatbot & Area-wise Access (`[SOURCE §21–§24]`)

- WhatsApp Chatbot for farmers (registered mobile identifies Farmer ID) — queries: Outstanding, Ledger, Purchases, Payments, Last Payment, Last Purchase, Invoice Details, Statement, Payment Status.
- Area-wise Farmer Allocation: farmers and employees assigned to areas; employees restricted to their assigned area at **backend/database authorization level**.
- Phase 2 Admin Functions: create/manage areas, assign farmers, assign employees, reassign farmers, transfer employees, view area-wise performance.
- Phase 2 Reporting: area-wise farmer count / procurement / payment / outstanding; employee-wise performance (farmers handled, invoices created, quantity purchased, purchase value, average rate); corrections and cancellations.

### 8.3 Future Scope

- Scale to 50,000 farmers and 500 employees without fundamental redevelopment (`[SOURCE §19]`).
- Activation of future-ready fields as the business grows: Area, Collection centre, GPS, Quality/grade, Lot number, Batch number, Warehouse, Packing, Export shipment (`[SOURCE §20]`).
- `[NEW]` Farmer Portal extension in later phases without a mobile-app dependency.

---

## 9. Detailed Feature Requirements

### 9.1 Requirement ID conventions

- Prefixes: `PRD-FRM` (Farmer Master), `PRD-EMP` (Employee Master), `PRD-PRT` (Farmer Portal `[NEW]`), `PRD-PROC` (Procurement), `PRD-INV` (Invoice), `PRD-PAY` (Payment & Banking), `PRD-LED` (Ledger), `PRD-MST` (Monthly Statement), `PRD-WH` (WhatsApp), `PRD-ADM` (Admin), `PRD-RPT` (Reporting), `PRD-AUD` (Audit), `PRD-SEC` (Security), `PRD-SCL` (Scalability), `PRD-NFR` (Non-functional), `PRD-BR` (Business rules), `PRD-SUC` (Success criteria).
- Detailed requirements for each module live in §10–§21; §9 below covers the two master-data modules and a module-to-section traceability map.

### 9.2 Farmer Master requirements (`[SOURCE §3]`)

| ID | Requirement | Priority | Source |
|---|---|---|---|
| PRD-FRM-001 | Each farmer MUST receive a permanent, unique Farmer ID (e.g., F-0001, F-0002, F-0003). | MUST | `[SOURCE §3]` |
| PRD-FRM-002 | The Farmer ID MUST be the permanent primary business identity of the farmer. | MUST | `[SOURCE §3]` |
| PRD-FRM-003 | Farmer Master MUST capture: Farmer ID, farmer name, mobile number, address, village, taluka, district, state, bank name, account number, IFSC, KYC information/documents where required, registration date, status, products normally supplied, and internal remarks. | MUST | `[SOURCE §3]` |
| PRD-FRM-004 | The registered mobile number MUST be linked to the Farmer ID for WhatsApp identification. | MUST | `[SOURCE §3]` |
| PRD-FRM-005 | Bank/tax/sensitive fields MUST be subject to restricted access (see PRD-SEC-014). | MUST | `[SOURCE §18]` |
| PRD-FRM-006 | KYC document types and validation level. Not specified in the source requirements. (Open question OQ-10.) | SHOULD | — |
| PRD-FRM-007 | Farmer "status" values and permitted transitions. Not specified in the source requirements. | SHOULD | — |
| PRD-FRM-008 | Deduplication policy when the same mobile/WhatsApp number maps to more than one Farmer ID. Not specified in the source requirements. (Open question OQ-13.) | SHOULD | — |

### 9.3 Employee Master requirements (`[SOURCE §4]`)

| ID | Requirement | Priority | Source |
|---|---|---|---|
| PRD-EMP-001 | Each employee MUST receive a unique Employee ID (e.g., EMP-001, EMP-002). | MUST | `[SOURCE §4]` |
| PRD-EMP-002 | Employee Master MUST capture: name, mobile number, email, authentication credentials, role and status. | MUST | `[SOURCE §4]` |
| PRD-EMP-003 | Area/location field MUST be reserved in the data model for Phase 2 — inactive in Phase 1. | MUST | `[SOURCE §4, §20]` |
| PRD-EMP-004 | The system MUST support the initial target of 100 employees and scale toward 500. | MUST | `[SOURCE §2, §19]` |
| PRD-EMP-005 | Every transaction MUST record the employee who created it (backend trail). | MUST | `[SOURCE §2]` |
| PRD-EMP-006 | Full role/permission catalogue beyond the two documented roles. Not specified in the source requirements. (Open question OQ-09.) | SHOULD | — |

### 9.4 Module traceability

| Module | Requirements section |
|---|---|
| Farmer Master | §9.2 (PRD-FRM) |
| Employee Master | §9.3 (PRD-EMP) |
| Farmer Portal (`[NEW]`) | §10 (PRD-PRT) |
| Procurement | §11 (PRD-PROC) |
| Invoice | §12 (PRD-INV) |
| Payment & Banking | §13 (PRD-PAY) |
| Ledger | §14 (PRD-LED) |
| Monthly Statement | §15 (PRD-MST) |
| WhatsApp | §16 (PRD-WH) |
| Admin | §17 (PRD-ADM) |
| Reporting | §18 (PRD-RPT) |
| Audit | §19 (PRD-AUD) |
| Security | §20 (PRD-SEC) |
| Scalability | §21 (PRD-SCL) |

---

## 10. Farmer Portal Requirements

> **Status: `[NEW]` — newly approved requirement (index N-01, N-02).** Not part of the original PDF. The Farmer Portal MUST be responsive and web/mobile-browser based in Phase 1; a separate native mobile application is out of scope in Phase 1 (`[SOURCE §1]`, reaffirmed).

| ID | Requirement | Priority | Source / note |
|---|---|---|---|
| PRD-PRT-001 | The system MUST provide a Farmer Login. | MUST | `[NEW]` approved |
| PRD-PRT-002 | The system MUST provide a Farmer Web Portal for farmers to access their own information. | MUST | `[NEW]` approved |
| PRD-PRT-003 | The portal MUST be responsive (usable on desktop and mobile web browsers). | MUST | `[NEW]` approved; N-03 boundary |
| PRD-PRT-004 | The portal MUST NOT be delivered as a native mobile application in Phase 1. | MUST | `[SOURCE §1]`, N-03 |
| PRD-PRT-005 | The portal MAY reuse the registered mobile number as the farmer identity anchor. | SHOULD | Consistent with `[SOURCE §3, §21]`; proposed mapping for portal logins. |
| PRD-PRT-006 | **Authentication method.** A secure login is required; **OTP-based authentication via the registered mobile number is a Proposed Enhancement** — implementation detail not specified in the source requirements. Final method pending OQ-02. | SHOULD | `[NEW]`; proposed |
| PRD-PRT-007 | The portal MUST show a **Dashboard** summarising the farmer's own position (e.g., recent purchases, recent payments, outstanding). Exact widgets. Not specified in the source requirements. | SHOULD | `[NEW]` |
| PRD-PRT-008 | The portal MUST provide **My Profile** showing the farmer's own master data (identity, contact, village/taluka/district/state, bank details, KYC status). What the farmer may view/edit beyond read-only. Not specified in the source requirements. | SHOULD | `[NEW]` |
| PRD-PRT-009 | The portal MUST provide **My Purchases** (list of the farmer's own procurement transactions). | SHOULD | `[NEW]` |
| PRD-PRT-010 | The portal MUST provide **My Invoices** (list of the farmer's own invoices with status). | SHOULD | `[NEW]` |
| PRD-PRT-011 | The portal MUST provide **My Payments** (list of the farmer's own payments incl. date, mode, UTR and status). | SHOULD | `[NEW]` |
| PRD-PRT-012 | The portal MUST provide **My Ledger** (the farmer's own ledger view per PRD-LED). | SHOULD | `[NEW]` |
| PRD-PRT-013 | The portal MUST provide **My Statements** (the farmer's own monthly statements, incl. PDF view/download of statements generated per §15). | SHOULD | `[NEW]` |
| PRD-PRT-014 | The portal MUST provide **Notifications** showing the farmer's own WhatsApp notifications (purchase confirmation, payment confirmation, monthly statement). | SHOULD | `[NEW]` |
| PRD-PRT-015 | The portal MUST provide a **Support** entry point (contact channel). The support channel/method. Not specified in the source requirements. | SHOULD | `[NEW]` |
| PRD-PRT-016 | A farmer MUST be restricted to their own data only; access to other farmers' data MUST NOT be possible. | MUST | `[NEW]`; derived from `[SOURCE §22]` philosophy (enforced at backend/database level) |
| PRD-PRT-017 | Portal content MUST reflect the system of record (purchases, invoices, payments, ledger, statements) in real time or near real time. | SHOULD | `[NEW]` |
| PRD-PRT-018 | Portal language(s) and number/currency display (₹). Not specified in the source requirements. (Open question OQ-17.) | SHOULD | — |
| PRD-PRT-019 | Farmer portal account activation/onboarding flow. Not specified in the source requirements. (Open question OQ-15.) | SHOULD | — |
| PRD-PRT-020 | Which portal data (e.g., bank details, KYC) a farmer may view or edit. Not specified in the source requirements. (Open question OQ-18.) | SHOULD | — |

---

## 11. Procurement Requirements (`[SOURCE §5]`)

| ID | Requirement | Priority | Source |
|---|---|---|---|
| PRD-PROC-001 | Procurement MUST follow the workflow: Login → Select/Search Farmer → Select Product → Enter Weight → Enter Rate → Calculate Amount → Confirm → Generate Invoice. | MUST | `[SOURCE §5]` |
| PRD-PROC-002 | A procurement transaction MUST record: Farmer ID, farmer name, Employee ID, date and time, product, quantity/weight, unit, rate, gross amount, deduction (if applicable), net amount, remarks. | MUST | `[SOURCE §5]` |
| PRD-PROC-003 | Amount MUST be calculated automatically as Quantity × Rate. | MUST | `[SOURCE §5]` |
| PRD-PROC-004 | The employee who created the transaction MUST be recorded (backend trail). | MUST | `[SOURCE §2, §5]` |
| PRD-PROC-005 | The employee MUST be able to select/search a farmer to begin entry. | MUST | `[SOURCE §5]` |
| PRD-PROC-006 | Deduction types (flat/percentage), reasons and approval rules. Not specified in the source requirements. (Open question OQ-06.) | SHOULD | — |
| PRD-PROC-007 | Product/unit master (managed catalogue), rate entry rules and approval for rate deviations. Not specified in the source requirements. (Open question OQ-05.) | SHOULD | — |
| PRD-PROC-008 | For a procurement employee, access to farmers MUST be limited to the farmer information required for procurement plus procurement entry (`[SOURCE §16]`). | MUST | `[SOURCE §16]` |

---

## 12. Invoice Requirements (`[SOURCE §6, §7]`)

| ID | Requirement | Priority | Source |
|---|---|---|---|
| PRD-INV-001 | Invoice numbers MUST be generated automatically from the Farmer ID and the transaction sequence (e.g., F-0001 → F-0001-01, F-0001-02, …). | MUST | `[SOURCE §6]` |
| PRD-INV-002 | Farmer ID and transaction sequence MUST be stored as separate database fields. | MUST | `[SOURCE §6]` |
| PRD-INV-003 | Every invoice number MUST be unique. | MUST | `[SOURCE §6]` |
| PRD-INV-004 | The sequence MUST be maintained separately for every farmer. | MUST | `[SOURCE §6]` |
| PRD-INV-005 | Cancelled invoices MUST remain in the system. | MUST | `[SOURCE §6]` |
| PRD-INV-006 | Cancelled invoice numbers MUST NOT be reused. | MUST | `[SOURCE §6]` |
| PRD-INV-007 | All modifications to invoices MUST be recorded in the audit log. | MUST | `[SOURCE §6, §17]` |
| PRD-INV-008 | Employees MUST NOT manually type invoice numbers. | MUST | `[SOURCE §6]` |
| PRD-INV-009 | On successful generation, the system MUST display the transaction confirmation with: Farmer, Invoice, Date, Product, Quantity, Rate, Total, Employee. | MUST | `[SOURCE §7]` |
| PRD-INV-010 | The confirmation SHOULD be printable and/or a downloadable PDF invoice; invoice output/reprint rules. Not specified in the source requirements. (Open question OQ-07.) | SHOULD | — |
| PRD-INV-011 | Cancellation workflow depth (who may cancel, reason capture, ledger impact). Detail not specified in the source requirements. (Open question OQ-08.) | SHOULD | — |

---

## 13. Payment & Banking Requirements (`[SOURCE §10, §11]`)

| ID | Requirement | Priority | Source |
|---|---|---|---|
| PRD-PAY-001 | The payment module MUST capture: Farmer ID, invoice number / invoice allocation, payment amount, payment date, payment status, payment mode, bank reference, UTR, remarks. | MUST | `[SOURCE §10]` |
| PRD-PAY-002 | The system MUST support full payment against an invoice. | MUST | `[SOURCE §10]` |
| PRD-PAY-003 | The system MUST support partial payment against an invoice. | MUST | `[SOURCE §10]` |
| PRD-PAY-004 | The system MUST support multiple payments against one invoice. | MUST | `[SOURCE §10]` |
| PRD-PAY-005 | The system MUST support multiple invoices against one payment. | MUST | `[SOURCE §10]` |
| PRD-PAY-006 | Banking integration MUST follow the workflow: Portal → Bank/API → Payment → UTR/status → Portal. | MUST | `[SOURCE §11]` |
| PRD-PAY-007 | Wherever supported by the selected banking/API provider, the system MUST automatically receive payment status and UTR. | MUST | `[SOURCE §11]` |
| PRD-PAY-008 | The system MUST update the farmer ledger automatically when payment is confirmed. | MUST | `[SOURCE §11]` |
| PRD-PAY-009 | The system MUST maintain payment statuses: matched, unmatched, failed, pending and duplicate. | MUST | `[SOURCE §11]` |
| PRD-PAY-010 | The system MUST provide an Exception/Reconciliation Queue for accounts staff. | MUST | `[SOURCE §11]` |
| PRD-PAY-011 | Reconciliation handling by accounts staff (roles/permissions). Not specified in the source requirements. | SHOULD | — |
| PRD-PAY-012 | Banking provider and integration mode (push/pull, webhook/polling, UTR timing). Not specified in the source requirements. (Open question OQ-03.) | SHOULD | — |
| PRD-PAY-013 | Payment mode values (e.g., bank transfer, etc.). Not specified in the source requirements. | SHOULD | — |

---

## 14. Ledger Requirements (`[SOURCE §9]`)

| ID | Requirement | Priority | Source |
|---|---|---|---|
| PRD-LED-001 | The system MUST maintain a complete farmer-wise ledger. | MUST | `[SOURCE §9]` |
| PRD-LED-002 | The ledger MUST show the farmer's transactions with date, invoice number, product, quantity, rate, amount, payment status and UTR number. | MUST | `[SOURCE §9]` |
| PRD-LED-003 | The ledger MUST be updated automatically upon confirmed payments (see PRD-PAY-008). | MUST | `[SOURCE §11]` |
| PRD-LED-004 | Outstanding balance MUST be derivable from the ledger per farmer. | MUST | Derived from `[SOURCE §9, §14, §15]` (outstanding is an established reportable/dashboard quantity). |
| PRD-LED-005 | Treatment of cancelled invoices in the ledger. Not specified in the source requirements. (Open question OQ-08.) | SHOULD | — |

---

## 15. Monthly Statement Requirements (`[SOURCE §13]`)

| ID | Requirement | Priority | Source |
|---|---|---|---|
| PRD-MST-001 | On the 1st day of every month, the system MUST automatically generate the statement for the previous month (e.g., on 1 September generate 1–31 August). | MUST | `[SOURCE §13]` |
| PRD-MST-002 | The statement MUST include: Farmer ID and farmer name, statement period, opening balance (if applicable), all purchase invoices (product, quantity, rate and invoice amount), all payments (payment dates and UTRs), closing/outstanding balance. | MUST | `[SOURCE §13]` |
| PRD-MST-003 | The statement MUST be generated as a PDF. | MUST | `[SOURCE §13]` |
| PRD-MST-004 | The statement MUST be sent to the farmer automatically through WhatsApp. | MUST | `[SOURCE §13]` |
| PRD-MST-005 | The system MUST record statement statuses: generated, sent, delivered, failed, retry. | MUST | `[SOURCE §13]` |
| PRD-MST-006 | Exact generation time/timezone and retry policy details. Not specified in the source requirements. (Open question OQ-14.) | SHOULD | — |

---

## 16. WhatsApp Requirements (`[SOURCE §8, §12, §13, §21]`)

| ID | Requirement | Priority | Source |
|---|---|---|---|
| PRD-WH-001 | After a successful invoice creation, the system MUST automatically send a WhatsApp purchase notification to the farmer. | MUST | `[SOURCE §8]` |
| PRD-WH-002 | The purchase notification MUST convey the purchase confirmation details (e.g., invoice number, date, product, quantity, rate, total). | MUST | `[SOURCE §8]` |
| PRD-WH-003 | When the bank/API confirms a payment, the system MUST automatically send the farmer a WhatsApp payment notification with the amount and UTR. | MUST | `[SOURCE §12]` |
| PRD-WH-004 | The farmer is identified on WhatsApp by the registered mobile number linked to the Farmer ID. | MUST | `[SOURCE §3, §21]` |
| PRD-WH-005 | The monthly statement MUST be delivered via WhatsApp (see PRD-MST-004). | MUST | `[SOURCE §13]` |
| PRD-WH-006 | WhatsApp provider and message policy (utility vs template) for business-initiated messages. Not specified in the source requirements. (Open question OQ-04.) | SHOULD | — |
| PRD-WH-007 | Failure handling/retry for purchase and payment notifications. Not specified in the source requirements (statement retry status exists; others undefined). | SHOULD | — |
| PRD-WH-008 | **Phase 2:** a WhatsApp Chatbot MUST let farmers query via WhatsApp: My Outstanding, My Ledger, My Purchases, My Payments, Last Payment, Last Purchase, Invoice Details, Statement, Payment Status. | MUST (Phase 2) | `[SOURCE §21]` |

---

## 17. Admin Requirements (`[SOURCE §2, §15, §16, §23]`)

| ID | Requirement | Priority | Source |
|---|---|---|---|
| PRD-ADM-001 | Super Admin MUST have full access to the entire system. | MUST | `[SOURCE §2]` |
| PRD-ADM-002 | Super Admin MUST be able to manage employees and farmers. | MUST | `[SOURCE §2]` |
| PRD-ADM-003 | Super Admin MUST be able to view all areas, procurement, invoices, payments, ledgers and reports. | MUST | `[SOURCE §2]` |
| PRD-ADM-004 | Super Admin MUST be able to manage settings, WhatsApp and banking integrations. | MUST | `[SOURCE §2]` |
| PRD-ADM-005 | Super Admin MUST be able to view audit logs. | MUST | `[SOURCE §2, §17]` |
| PRD-ADM-006 | The Admin Dashboard MUST show: total farmers, active farmers, total employees, today's procurement, today's procurement value, monthly procurement, pending farmer payments, payments made, total outstanding, number of invoices, unreconciled bank transactions. | MUST | `[SOURCE §15]` |
| PRD-ADM-007 | The Admin Dashboard MUST support date filters. | MUST | `[SOURCE §15]` |
| PRD-ADM-008 | The permission matrix MUST be configurable by Admin. | MUST | `[SOURCE §16]` |
| PRD-ADM-009 | **Phase 2:** Admin MUST be able to create and manage areas. | MUST (Phase 2) | `[SOURCE §23]` |
| PRD-ADM-010 | **Phase 2:** Admin MUST be able to assign farmers and employees to areas, reassign farmers and transfer employees. | MUST (Phase 2) | `[SOURCE §23]` |
| PRD-ADM-011 | **Phase 2:** Admin MUST be able to view area-wise performance; full Admin/Management access maintained. | MUST (Phase 2) | `[SOURCE §23]` |

---

## 18. Reporting Requirements (`[SOURCE §14, §24]`)

| ID | Requirement | Priority | Source |
|---|---|---|---|
| PRD-RPT-001 | Farmer reports MUST include farmer-wise purchase, payment, outstanding, ledger and monthly statement. | MUST | `[SOURCE §14]` |
| PRD-RPT-002 | Procurement reports MUST support date-wise, product-wise, employee-wise, farmer-wise, quantity-wise and rate-wise views. | MUST | `[SOURCE §14]` |
| PRD-RPT-003 | Payment reports MUST include payment register, UTR register, paid, partially paid, pending and unreconciled payments. | MUST | `[SOURCE §14]` |
| PRD-RPT-004 | Reports MUST be exportable to Excel, PDF and CSV where appropriate. | MUST | `[SOURCE §14]` |
| PRD-RPT-005 | **Phase 2:** reporting MUST include area-wise farmer count, area-wise procurement, area-wise payment, area-wise outstanding, and employee-wise performance (farmers handled, invoices created, quantity purchased, purchase value, average rate); plus corrections and cancellations scope. | MUST (Phase 2) | `[SOURCE §24]` |
| PRD-RPT-006 | Report field definitions, filters and scheduling. Not specified in the source requirements. | SHOULD | — |

---

## 19. Audit Requirements (`[SOURCE §17]`)

| ID | Requirement | Priority | Source |
|---|---|---|---|
| PRD-AUD-001 | The audit trail MUST record: user/employee ID, date and time, action, original value, new value, record affected. | MUST | `[SOURCE §17]` |
| PRD-AUD-002 | IP/device information MUST be recorded where appropriate. | MUST | `[SOURCE §17]` |
| PRD-AUD-003 | All invoice modifications MUST be recorded (see PRD-INV-007). | MUST | `[SOURCE §6]` |
| PRD-AUD-004 | Historical financial records MUST NOT be silently overwritten. | MUST | `[SOURCE §17]` |
| PRD-AUD-005 | Super Admin MUST be able to view audit logs (see PRD-ADM-005). | MUST | `[SOURCE §2]` |
| PRD-AUD-006 | Audit retention period and export/archival format. Not specified in the source requirements. (Open question OQ-11.) | SHOULD | — |

---

## 20. Security Requirements (`[SOURCE §18]`)

| ID | Requirement | Priority | Source |
|---|---|---|---|
| PRD-SEC-001 | The system MUST provide secure authentication. | MUST | `[SOURCE §18]` |
| PRD-SEC-002 | The system MUST enforce a strong password policy. | MUST | `[SOURCE §18]` |
| PRD-SEC-003 | The system MUST support OTP/2FA where appropriate. | MUST | `[SOURCE §18]` |
| PRD-SEC-004 | Sensitive data MUST be encrypted. | MUST | `[SOURCE §18]` |
| PRD-SEC-005 | Data MUST be encrypted in transit. | MUST | `[SOURCE §18]` |
| PRD-SEC-006 | The system MUST implement role-based authorization. | MUST | `[SOURCE §18]` |
| PRD-SEC-007 | API access MUST use secure API authentication. | MUST | `[SOURCE §18]` |
| PRD-SEC-008 | The system MUST perform regular automated database backups. | MUST | `[SOURCE §18]` |
| PRD-SEC-009 | Backup restoration MUST be tested regularly. | MUST | `[SOURCE §18]` |
| PRD-SEC-010 | The system MUST provide audit logging (see §19). | MUST | `[SOURCE §18]` |
| PRD-SEC-011 | The system MUST manage sessions (session management). | MUST | `[SOURCE §18]` |
| PRD-SEC-012 | The system MUST apply rate limiting. | MUST | `[SOURCE §18]` |
| PRD-SEC-013 | The system MUST protect against common web/API attacks. | MUST | `[SOURCE §18]` |
| PRD-SEC-014 | Bank details and other sensitive farmer information MUST have restricted access. | MUST | `[SOURCE §18]` |
| PRD-SEC-015 | **Phase 2:** area-based restrictions MUST be enforced at backend/database authorization level. | MUST (Phase 2) | `[SOURCE §22]` |
| PRD-SEC-016 | Farmer portal login MUST meet the same security baseline (authentication, session, rate limiting). | MUST | `[NEW]`; derived from `[SOURCE §18]` |

---

## 21. Scalability Requirements (`[SOURCE §1, §19]`)

| ID | Requirement | Priority | Source |
|---|---|---|---|
| PRD-SCL-001 | The system MUST support an initial target of 10,000 farmers and 100 employees. | MUST | `[SOURCE §1]` |
| PRD-SCL-002 | The architecture MUST scale toward 50,000 farmers and 500 employees without fundamental redevelopment. | MUST | `[SOURCE §1, §19]` |
| PRD-SCL-003 | The system MUST support concurrent employee usage. | MUST | `[SOURCE §19]` |
| PRD-SCL-004 | The database MUST support potentially millions of transaction records. | MUST | `[SOURCE §19]` |
| PRD-SCL-005 | The design MUST account for transaction volume, not just farmer count. | MUST | `[SOURCE §19]` |
| PRD-SCL-006 | The architecture SHOULD allow later integration of future-ready fields without rework: Area, Collection centre, GPS, Quality/grade, Lot number, Batch number, Warehouse, Packing, Export shipment. | MUST | `[SOURCE §20]` |
| PRD-SCL-007 | Future-ready fields MUST remain inactive placeholders in Phase 1. | MUST | `[SOURCE §20]` |
| PRD-SCL-008 | Performance targets at target scale (e.g., response times). Not specified in the source requirements. (Proposed Enhancement — see OQ list.) | SHOULD | — |

---

## 22. Non-Functional Requirements

| ID | Requirement | Priority | Source / note |
|---|---|---|---|
| PRD-NFR-001 | The system MUST be a web-based portal. | MUST | `[SOURCE §1]` |
| PRD-NFR-002 | The system MUST scale as defined in §21 (PRD-SCL-001…005). | MUST | `[SOURCE §1, §19]` |
| PRD-NFR-003 | The architecture MUST avoid fundamental redesign at target scale. | MUST | `[SOURCE §19]` |
| PRD-NFR-004 | All security requirements in §20 apply globally. | MUST | `[SOURCE §18]` |
| PRD-NFR-005 | An agreed availability/uptime target. Not specified in the source requirements. (Proposed Enhancement.) | SHOULD | — |
| PRD-NFR-006 | Operational recovery: automated backups + restoration testing (see PRD-SEC-008/009). | MUST | `[SOURCE §18]` |
| PRD-NFR-007 | Detailed performance (latency, throughput) and capacity targets. Not specified in the source requirements. (Proposed Enhancement — see OQ list.) | SHOULD | — |
| PRD-NFR-008 | Farmer Portal responsiveness on mobile browsers. | MUST | `[NEW]` (N-02/N-03) |

---

## 23. Business Rules

Rules below are stated in the source and MUST NOT be altered or removed. Anything else is undefined (see §28).

| ID | Business rule | Source |
|---|---|---|
| PRD-BR-001 | Farmer ID is permanent, unique, and is the farmer's primary business identity (format example: F-0001). | `[SOURCE §3]` |
| PRD-BR-002 | The registered mobile number is linked to the Farmer ID and is the WhatsApp identity anchor. | `[SOURCE §3, §21]` |
| PRD-BR-003 | Invoice numbers are generated automatically as Farmer ID + transaction sequence (e.g., F-0001-17); Farmer ID and sequence are stored as separate fields. | `[SOURCE §6]` |
| PRD-BR-004 | Invoice numbers must be unique; a per-farmer sequence is maintained separately for every farmer. | `[SOURCE §6]` |
| PRD-BR-005 | Cancelled invoices remain in the system and their numbers are never reused. | `[SOURCE §6]` |
| PRD-BR-006 | Employees never type invoice numbers manually; all invoice modifications are audited. | `[SOURCE §6, §17]` |
| PRD-BR-007 | Invoice amount equals Quantity × Rate, calculated automatically. | `[SOURCE §5]` |
| PRD-BR-008 | Every transaction records the employee who created it (backend trail). | `[SOURCE §2]` |
| PRD-BR-009 | Monthly statements are generated on the 1st of each month for the previous month. | `[SOURCE §13]` |
| PRD-BR-010 | Historical financial records must never be silently overwritten (corrections recorded via audit). | `[SOURCE §17]` |
| PRD-BR-011 | Payments support full, partial, multiple payments per invoice and multiple invoices per payment. | `[SOURCE §10]` |
| PRD-BR-012 | Payment statuses tracked: matched, unmatched, failed, pending, duplicate. | `[SOURCE §11]` |
| PRD-BR-013 | (Phase 2) Employees may only access farmers in their assigned area; enforced at backend/database level. | `[SOURCE §22]` |
| PRD-BR-014 | (Phase 1) No native mobile application for farmers. | `[SOURCE §1]`, N-03 |

---

## 24. Success Criteria

| ID | Success criterion | Derived from |
|---|---|---|
| PRD-SUC-001 | All invoices are generated without any manual invoice-number entry by employees. | `[SOURCE §6]` |
| PRD-SUC-002 | Farmer ledger updates automatically when a bank/API-confirmed payment is received. | `[SOURCE §11]` |
| PRD-SUC-003 | Monthly statements are automatically generated and delivered on the 1st with recorded delivery status (generated/sent/delivered/failed/retry). | `[SOURCE §13]` |
| PRD-SUC-004 | Farmers automatically receive WhatsApp purchase and payment notifications. | `[SOURCE §8, §12]` |
| PRD-SUC-005 | Phase 1 serves 10,000 farmers and 100 employees without operational failure. | `[SOURCE §1]` |
| PRD-SUC-006 | Architecture is proven to scale to 50,000 farmers / 500 employees without fundamental redesign. | `[SOURCE §19]` |
| PRD-SUC-007 | Farmers can log in to a responsive web portal and view their own data (`[NEW]`). | N-01/N-02 |
| PRD-SUC-008 | Reconciliation queue exposes unmatched/failed/duplicate transactions for accounts staff. | `[SOURCE §11]` |

---

## 25. Assumptions

Assumptions are working premises for design and delivery; they are **not** source requirements and MAY be revisited.

| ID | Assumption |
|---|---|
| ASM-001 | The business operates in INR (₹); source examples and statements use ₹ (e.g., ₹28/kg, ₹11,200). |
| ASM-002 | Procurement employees have internet-connected devices at collection points. Not specified in the source requirements. |
| ASM-003 | Farmers for whom WhatsApp notifications are sent have opted in / consented to receive messages on their registered numbers. Not specified in the source requirements. |
| ASM-004 | The banking/API provider exposes payment status and UTR programmatically (fully or partially), per `[SOURCE §11]` ("wherever supported"). |
| ASM-005 | Farmer portal users possess a mobile device or access to a web browser; portal is therefore responsive, not a native app (N-03). |
| ASM-006 | A valid WhatsApp number is a required precondition for notifications to succeed; failed numbers surface in statuses (e.g., failed/retry). |
| ASM-007 | Statutory details (GST, levies) are outside current business rules until confirmed (see OQ-12). |

---

## 26. Constraints

| ID | Constraint | Source |
|---|---|---|
| CON-001 | Farmers do not receive a separate native mobile application in Phase 1. | `[SOURCE §1]`, N-03 |
| CON-002 | Farmer Portal in Phase 1 is responsive web/mobile-browser based. | `[NEW]` (N-02) |
| CON-003 | Architecture must reach 50,000 farmers / 500 employees without fundamental redevelopment. | `[SOURCE §1, §19]` |
| CON-004 | Database must handle millions of transaction records. | `[SOURCE §19]` |
| CON-005 | Personal/statutory/regulatory compliance requirements beyond those in `[SOURCE §18]` are not yet defined (see OQ-12). |
| CON-006 | No application code is produced during the current documentation-preparation phase (index rule N-04). |

---

## 27. Out of Scope

| # | Item | Basis |
|---|---|---|
| 1 | Native mobile application for farmers in Phase 1 | `[SOURCE §1]`, N-03 |
| 2 | WhatsApp Chatbot (farmer queries over WhatsApp) | `[SOURCE §21]` — Phase 2 |
| 3 | Area-wise farmer/employee allocation and area restrictions | `[SOURCE §22–§24]` — Phase 2 |
| 4 | Activation of future-ready operational fields (Area, Collection centre, GPS, Quality/grade, Lot, Batch, Warehouse, Packing, Export shipment) | `[SOURCE §20]` — placeholders only in Phase 1 |
| 5 | Export/shipment and packing operations management | `[SOURCE §20]` — future |
| 6 | Tax/statutory computation (GST, levies) | Not specified in the source requirements (OQ-12) |
| 7 | Online/marketplace selling, inventory management, direct farmer-to-buyer transactions | Not specified in the source requirements |
| 8 | Multi-currency handling | Not specified in the source requirements |
| 9 | Any Farmer Portal marketing/engagement features | Not specified in the source requirements |

---

## 28. Open Questions

Open questions from the documentation index (`docs/00_DOCUMENTATION_INDEX.md`, §15), plus portal-specific items. All items remain open until decided; decisions are logged in `14_OPEN_QUESTIONS_AND_DECISIONS.md`.

| ID | Item | Origin |
|---|---|---|
| OQ-01 | Farmer Portal delivery phase (Phase 1 vs Phase 2) and full feature list | `[NEW]` — not specified in the source requirements |
| OQ-02 | Farmer authentication method (OTP on registered mobile vs credentials vs both) | `[NEW]` — not specified |
| OQ-03 | Banking/API provider and integration mode (push/pull, webhook/polling, UTR timing) | Not specified in the source requirements |
| OQ-04 | WhatsApp provider and messaging policy (utility vs template) | Not specified in the source requirements |
| OQ-05 | Product master (catalogue, units, rate rules, deviation approval) | Not specified in the source requirements |
| OQ-06 | Deduction rules (types, reasons, approvals, ledger impact) | Not specified in the source requirements |
| OQ-07 | Invoice output format (confirm-as-PDF, reprint rules) | Not specified in the source requirements |
| OQ-08 | Invoice cancellation workflow depth (authorization, reason, ledger impact) | Partially specified in the source requirements |
| OQ-09 | Employee permission catalogue beyond the two documented roles | Partially specified in the source requirements |
| OQ-10 | KYC specifics (documents, validation, status lifecycle) | Not specified in the source requirements |
| OQ-11 | Audit retention and export/archival format | Not specified in the source requirements |
| OQ-12 | Tax / statutory handling (GST, levies, commissions) | Not specified in the source requirements |
| OQ-13 | Farmer identity deduplication (same mobile → multiple Farmer IDs) | Not specified in the source requirements |
| OQ-14 | Statement generation timing/timezone and retry policy details | Partially specified in the source requirements |
| OQ-15 | Farmer portal account activation / onboarding flow | `[NEW]` — not specified |
| OQ-16 | Farmer portal support channel/method (PRD-PRT-015) | `[NEW]` — not specified |
| OQ-17 | Farmer portal language(s) and number/currency display | `[NEW]` — not specified |
| OQ-18 | Which portal data a farmer may view/edit (bank details, KYC) | `[NEW]` — not specified |

---

*End of Product Requirements Document v1.0. Next artifact in sequence: `02_BUSINESS_PROCESSES.md` (per index reading order).*