# Agri Procurement & Farmer Management System
## Module Specification

---

## 1. Document Information

| Item | Value |
|---|---|
| Product name | Agri Procurement & Farmer Management System |
| Document | Module Specification |
| Version | v1.0 |
| Status | Draft — aligned to PRD v1.0, BRD v1.0, Business Workflows v1.0 |
| Date | 2026-09-16 |
| Purpose | Identify and specify every system module: purpose, actors, responsibilities, features, inputs, outputs, business rules, dependencies, permissions, data, notifications, audit and phase |
| Source | 1. `AgriProcurement & Farmer Management.pdf` (primary source of truth) 2. `docs/00_DOCUMENTATION_INDEX.md` 3. `docs/01_PRODUCT_REQUIREMENTS_DOCUMENT.md` 4. `docs/02_BUSINESS_REQUIREMENTS_DOCUMENT.md` 5. `docs/03_BUSINESS_WORKFLOWS.md` |

### Tagging conventions

| Tag | Meaning |
|---|---|
| `[SOURCE §n]` | Requirement stated in the original PDF section `n` |
| `[NEW]` | Newly approved Farmer Portal requirement |
| Proposed Enhancement | Suggested addition; not yet approved |
| Undefined / Not specified in the source requirements | Behaviour not defined in the source; tracked in PRD §28 / BRD §25 |

### Phase legend

| Phase | Meaning |
|---|---|
| **Phase 1** | Core procurement, payment, statement system + Farmer Portal `[NEW]` |
| **Phase 2** | WhatsApp Chatbot & area-wise access |
| **Future / Proposed** | Capability not yet committed; suggested or deferred |

---

## 2. Module Index

| ID | Module | Phase |
|---|---|---|
| M01 | Authentication | Phase 1 |
| M02 | User Management | Phase 1 |
| M03 | Farmer Management | Phase 1 |
| M04 | Employee Management | Phase 1 |
| M05 | Farmer Portal | Phase 1 `[NEW]` |
| M06 | Procurement | Phase 1 |
| M07 | Product Management | Phase 1 (minimal) / undefined detail |
| M08 | Invoice Management | Phase 1 |
| M09 | Payment Management | Phase 1 |
| M10 | Bank Reconciliation | Phase 1 |
| M11 | Farmer Ledger | Phase 1 |
| M12 | Monthly Statements | Phase 1 |
| M13 | WhatsApp Integration | Phase 1 (outbound) + Phase 2 (chatbot) |
| M14 | Notifications | Phase 1 |
| M15 | Reports | Phase 1 (+ Phase 2 additions) |
| M16 | Dashboard | Phase 1 (+ Phase 2 additions) |
| M17 | Audit Logs | Phase 1 |
| M18 | System Settings | Phase 1 |
| M19 | WhatsApp Chatbot | Phase 2 |
| M20 | Area Management | Phase 2 |
| M21 | Area-based Farmer Allocation | Phase 2 |
| M22 | Area-based Employee Allocation | Phase 2 |
| M23 | Area-wise Reporting | Phase 2 |
| Future | Future-Ready Fields & proposed modules | Future / Proposed |

---

## 3. Module Specifications

---

### M01. Authentication — Phase 1

- **Purpose:** Securely identify users (Admin, Employee, and `[NEW]` Farmer) before access.
- **Actors:** System; Super Admin; Procurement Employee; Farmer (`[NEW]`).
- **Responsibilities:** Verify credentials; issue/manage sessions; enforce security controls.
- **Features:**
  - Secure authentication (`[SOURCE §18]`)
  - Strong password policy (`[SOURCE §18]`)
  - OTP/2FA where appropriate (`[SOURCE §18]`)
  - Session management (`[SOURCE §18]`)
  - Rate limiting (`[SOURCE §18]`)
  - Farmer portal authentication (`[NEW]`; OTP on registered mobile = Proposed Enhancement, method pending OQ-02)
  - Employee individual logins (`[SOURCE §2]`)
- **Inputs:** Credentials (password and/or OTP); session requests.
- **Outputs:** Authenticated session; access grant/deny.
- **Business rules:**
  - Every employee has an individual login (`[SOURCE §2]`).
  - Authentication must be secure; session and rate-limiting controls mandatory (`[SOURCE §18]`).
  - Farmer authentication method — Not specified in the source requirements (OQ-02).
- **Dependencies:** M02 (user accounts), M03 (farmer master for portal identity), M04 (employee records).
- **Permissions:** By user type via role-based authorization (`[SOURCE §18]`).
- **Data involved:** Credentials, session records.
- **Notifications:** OTP delivery if OTP scheme adopted (Proposed Enhancement).
- **Audit requirements:** Authentication/session events recorded (audit + session management `[SOURCE §17, §18]`).
- **Phase:** Phase 1.

---

### M02. User Management — Phase 1

- **Purpose:** Manage the identity records and accounts that users authenticate with.
- **Actors:** System; Super Admin; Farmer (`[NEW]`).
- **Responsibilities:** Account lifecycle (create, activate, disable); credential management; role/status maintenance.
- **Features:**
  - Create/disable user accounts (employee/admin)
  - Authentication credentials per user (`[SOURCE §4]`)
  - Role assignment (`[SOURCE §4, §16]`)
  - Status per user (`[SOURCE §4]`)
  - Farmer portal account activation (`[NEW]`; onboarding flow undefined OQ-15)
- **Inputs:** User identity data; role/status changes.
- **Outputs:** Usable accounts; permission profile.
- **Business rules:**
  - Permission matrix configurable by Admin (`[SOURCE §16]`).
  - Full role catalogue — Not specified in the source requirements (OQ-09).
- **Dependencies:** M01 (authentication), M04 (employee master), M03 (farmer master), M18 (settings).
- **Permissions:** Admin full access (`[SOURCE §2]`).
- **Data involved:** User accounts, roles, status, credential records.
- **Notifications:** None specified.
- **Audit requirements:** Account changes recorded (`[SOURCE §17]`).
- **Phase:** Phase 1.

---

### M03. Farmer Management — Phase 1

- **Purpose:** Maintain the Farmer Master — the permanent identity record of every farmer.
- **Actors:** Super Admin; Procurement Employee (restricted); System.
- **Responsibilities:** Farmer registration and updates; Farmer ID; KYC; bank data; mobile–WhatsApp linking; status; internal remarks.
- **Features:**
  - Farmer registration (BRD W01)
  - Permanent unique Farmer ID generation (BRD W02)
  - Fields: Farmer ID, name, mobile, address, village/taluka/district/state, bank name/account/IFSC, KYC where required, registration date, status, products normally supplied, internal remarks (`[SOURCE §3]`)
  - Search/select farmer for procurement
  - Mobile linked to Farmer ID for WhatsApp identification (`[SOURCE §3]`)
- **Inputs:** Farmer master data; KYC documents; bank details.
- **Outputs:** Farmer record; Farmer ID; WhatsApp-identifiable farmer identity.
- **Business rules:**
  - Farmer ID is permanent, unique, primary business identity (`[SOURCE §3]`).
  - Registered mobile is the WhatsApp identity anchor (`[SOURCE §3]`).
  - KYC specifics, status transitions, deduplication — Not specified in the source requirements (OQ-10, OQ-13).
- **Dependencies:** M02, M04 (admin/employee context), M06 (procurement reads farmer), M09 (payments to farmer bank), M13 (WhatsApp).
- **Permissions:** Admin manages farmers (`[SOURCE §2]`); procurement employee limited to farmer info needed for procurement (`[SOURCE §16]`); restricted access to sensitive bank data (`[SOURCE §18]`).
- **Data involved:** Farmer Master (PII + bank + KYC).
- **Notifications:** None at registration.
- **Audit requirements:** Master changes recorded (`[SOURCE §17]`).
- **Phase:** Phase 1.

---

### M04. Employee Management — Phase 1

- **Purpose:** Maintain the Employee Master.
- **Actors:** Super Admin; System.
- **Responsibilities:** Employee record lifecycle; credentials; role; status.
- **Features:**
  - Fields: Employee ID (EMP-001), name, mobile, email, authentication credentials, role, status (`[SOURCE §4]`)
  - Area/location field reserved for Phase 2 (`[SOURCE §4]`)
- **Inputs:** Employee data; role/status assignment.
- **Outputs:** Employee identity usable in login and transaction attribution.
- **Business rules:**
  - Every transaction records the creating employee (`[SOURCE §2]`).
  - Initial target 100 employees; architecture supports 500+ (`[SOURCE §2, §19]`).
- **Dependencies:** M02 (accounts), M01 (login), M06 (attribution).
- **Permissions:** Admin full access (`[SOURCE §2]`).
- **Data involved:** Employee Master.
- **Notifications:** None specified.
- **Audit requirements:** Employee changes recorded (`[SOURCE §17]`).
- **Phase:** Phase 1.

---

### M05. Farmer Portal — Phase 1 `[NEW]`

- **Purpose:** Farmer self-service web portal (`[NEW]` — newly approved requirement; not in original PDF).
- **Actors:** Farmer; System.
- **Responsibilities:** Farmer login; self-service view of own financial data; support contact.
- **Features:**
  - Farmer Login (`[NEW]`; OTP = Proposed Enhancement, OQ-02)
  - Dashboard — own summary (widgets undefined OQ-01)
  - My Profile — own master data (edit scope undefined OQ-18)
  - My Purchases — own purchases
  - My Invoices — own invoices + status
  - My Payments — own payments (date, mode, UTR, status)
  - My Ledger — own ledger view
  - My Statements — own statement PDFs
  - Notifications — own notification history
  - Support — contact channel (channel undefined OQ-16)
- **Inputs:** Farmer credentials; portal navigation.
- **Outputs:** Read-only views of own data.
- **Business rules:**
  - Responsive web/mobile-browser based; no native mobile app in Phase 1 (`[NEW]` N-03; `[SOURCE §1]`).
  - Farmer sees own data only, enforced at backend level (derived from `[SOURCE §22]` philosophy).
- **Dependencies:** M01 (auth `[NEW]`), M03 (farmer identity), M06/M08/M09/M11/M12/M13/M14 (data sources), M17 (audit).
- **Permissions:** Farmer — self only.
- **Data involved:** Read-only subset of farmer's own records.
- **Notifications:** Reflects notification history; no new outbound type specified.
- **Audit requirements:** View-action logging — Not specified in the source requirements (Proposed Enhancement if required).
- **Phase:** Phase 1.

---

### M06. Procurement — Phase 1

- **Purpose:** Record purchases of produce from farmers at collection time.
- **Actors:** Procurement Employee; System.
- **Responsibilities:** Guided purchase entry; automatic amount calculation; transaction attribution.
- **Features:**
  - Workflow: Login → Select/Search Farmer → Select Product → Enter Weight → Enter Rate → Calculate Amount → Confirm → Generate Invoice (`[SOURCE §5]`)
  - Fields: farmer ID, farmer name, employee ID, date/time, product, quantity/weight, unit, rate, gross amount, deduction (if applicable), net amount, remarks (`[SOURCE §5]`)
  - Automatic amount = quantity × rate (`[SOURCE §5]`)
- **Inputs:** Farmer selection, product, weight, rate, optional deduction, remarks.
- **Outputs:** Confirmed procurement transaction; input to invoicing.
- **Business rules:**
  - Amount computed automatically (`[SOURCE §5]`).
  - Deduction rules — Not specified in the source requirements (OQ-06).
  - Product/rate governance — see M07; undefined (OQ-05).
- **Dependencies:** M03 (farmer), M07 (product/unit), M08 (invoice trigger), M14 (notification trigger).
- **Permissions:** Procurement Employee (limited), Admin (full) (`[SOURCE §16]`).
- **Data involved:** Procurement transaction.
- **Notifications:** Purchase WhatsApp notification after invoicing (`[SOURCE §8]`).
- **Audit requirements:** Transaction creation + employee attribution recorded (`[SOURCE §2, §17]`).
- **Phase:** Phase 1.

---

### M07. Product Management — Phase 1 (minimal) / undefined detail

- **Purpose:** Provide the selectable product/unit reference for procurement.
- **Actors:** Super Admin; System (permission/scope undefined).
- **Responsibilities:** Maintain product options and units used during purchase entry.
- **Features:**
  - Product selection in procurement (`[SOURCE §5]`)
  - Unit per product (`[SOURCE §5]`)
- **Inputs:** Product definitions (specific fields undefined).
- **Outputs:** Product/unit choices for procurement.
- **Business rules:**
  - A product catalogue and unit master — Not specified in the source requirements (OQ-05).
  - Rate approval for deviations — Not specified in the source requirements (OQ-05).
  - Future-ready quality/grade, lot, batch — inactive Phase 1 `[SOURCE §20]` (Future).
- **Dependencies:** M06 (procurement).
- **Permissions:** Not specified in the source requirements (Proposed: Admin-managed catalogue).
- **Data involved:** Product/unit reference data.
- **Notifications:** None.
- **Audit requirements:** Master changes recorded (`[SOURCE §17]`).
- **Phase:** Phase 1 (minimum needed by procurement); full catalogue governance undefined.

---

### M08. Invoice Management — Phase 1

- **Purpose:** Automatic, unique, correctly numbered invoices with confirmation display.
- **Actors:** System (numbering); Procurement Employee; Admin.
- **Responsibilities:** Invoice generation; numbering per farmer sequence; cancellation retention; confirmation display.
- **Features:**
  - Auto numbering from Farmer ID + per-farmer sequence (e.g., F-0001-17) (`[SOURCE §6]`)
  - Farmer ID and sequence stored as separate fields (`[SOURCE §6]`)
  - Unique invoice numbers (`[SOURCE §6]`)
  - Per-farmer independent sequence (`[SOURCE §6]`)
  - Cancelled invoices retained; numbers never reused (`[SOURCE §6]`)
  - Confirmation display: Farmer, Invoice, Date, Product, Quantity, Rate, Total, Employee (`[SOURCE §7]`)
- **Inputs:** Confirmed procurement transaction.
- **Outputs:** Invoice record; confirmation; sequence incremented.
- **Business rules:**
  - Employees never type invoice numbers (`[SOURCE §6]`).
  - All invoice modifications audited (`[SOURCE §6, §17]`).
  - Cancellation actor/reason/ledger impact — Not specified (OQ-08).
  - Printable/PDF output — Not specified (OQ-07).
- **Dependencies:** M06 (transaction), M11 (posting), M14 (purchase notification trigger).
- **Permissions:** Creation tied to procuring employee; cancellation actor undefined (OQ-08).
- **Data involved:** Invoice records; invoice sequence.
- **Notifications:** Purchase WhatsApp notification (`[SOURCE §8]`).
- **Audit requirements:** Creation + all modifications logged (`[SOURCE §6, §17]`).
- **Phase:** Phase 1.

---

### M09. Payment Management — Phase 1

- **Purpose:** Record and allocate payments against farmer invoices.
- **Actors:** Payment creator (actor undefined — likely accounts/admin, Proposed); System; Accounts staff.
- **Responsibilities:** Payment records; allocation to invoices; statuses; bank reference/UTR.
- **Features:**
  - Fields: Farmer ID, invoice number/allocation, amount, date, status, mode, bank reference, UTR, remarks (`[SOURCE §10]`)
  - Full payment, partial payment, multiple payments per invoice, multiple invoices per payment (`[SOURCE §10]`)
- **Inputs:** Payment details + invoice allocation.
- **Outputs:** Payment record; updated allocations; hand-off to bank processing.
- **Business rules:**
  - Allocation modes per `[SOURCE §10]`.
  - Over-allocation prevention — Not specified (Proposed Enhancement).
  - Payment creator role — Not specified in the source requirements.
- **Dependencies:** M03 (farmer/bank), M08 (invoices), M10 (bank processing/reconciliation), M11 (ledger), M14 (payment notification).
- **Permissions:** Accounts/Admin implied (`[SOURCE §11]` queue for accounts staff); not formally specified.
- **Data involved:** Payment records, invoice allocations.
- **Notifications:** WhatsApp payment notification on confirmation (`[SOURCE §12]`).
- **Audit requirements:** Payment changes recorded (`[SOURCE §17]`).
- **Phase:** Phase 1.

---

### M10. Bank Reconciliation — Phase 1

- **Purpose:** Receive payment status/UTR and reconcile payments to invoices.
- **Actors:** System; Banking/API provider; Accounts staff.
- **Responsibilities:** Payment execution hand-off; status/UTR receipt; matching; exception queue.
- **Features:**
  - Portal → Bank/API → Payment → UTR/status → Portal (`[SOURCE §11]`)
  - Automatic UTR/status receipt wherever supported (`[SOURCE §11]`)
  - Statuses: matched, unmatched, failed, pending, duplicate (`[SOURCE §11]`)
  - Exception/Reconciliation Queue for accounts staff (`[SOURCE §11]`)
  - Automatic ledger update on match (`[SOURCE §11]`)
- **Inputs:** Bank/API responses (UTR/status).
- **Outputs:** Reconciled payments; exception queue items; ledger updates.
- **Business rules:**
  - Statuses mandated (`[SOURCE §11]`).
  - Provider and mode (push/pull, webhook/polling) — Not specified (OQ-03).
  - Queue resolution rules — Not specified (OQ-03 related).
- **Dependencies:** M09 (payments), M11 (ledger), M13 (provider connectivity), M18 (integration settings).
- **Permissions:** Accounts staff for the queue (`[SOURCE §11]`).
- **Data involved:** Payment statuses, UTR, reconciliation queue.
- **Notifications:** Payment WhatsApp notification on confirmation (`[SOURCE §12]`).
- **Audit requirements:** Reconciliation actions recorded (`[SOURCE §17]`); secure API auth (`[SOURCE §18]`).
- **Phase:** Phase 1.

---

### M11. Farmer Ledger — Phase 1

- **Purpose:** Maintain the complete farmer-wise ledger.
- **Actors:** System; Admin (read); Employee (read, scoped); Farmer via portal (read own).
- **Responsibilities:** Post invoices; update on confirmed payments; expose outstanding.
- **Features:**
  - Farmer-wise ledger (`[SOURCE §9]`)
  - Columns: date, invoice, product, quantity, rate, amount, payment, UTR (`[SOURCE §9]`)
  - Automatic update on confirmed payment (`[SOURCE §11]`)
- **Inputs:** Invoice postings; confirmed payments.
- **Outputs:** Up-to-date ledger; outstanding balance.
- **Business rules:**
  - Ledger is system-maintained; no silent overwrite (`[SOURCE §17]`).
  - Cancelled-invoice treatment — Not specified (OQ-08).
- **Dependencies:** M08 (invoices), M09/M10 (payments), M12 (statements), M05 (portal view).
- **Permissions:** Admin full (`[SOURCE §2]`); employee scoped (`[SOURCE §16]`); farmer own data (`[NEW]`).
- **Data involved:** Farmer ledger.
- **Notifications:** None direct (feeds statements/notifications).
- **Audit requirements:** Ledger changes traceable; no silent overwrite (`[SOURCE §17]`).
- **Phase:** Phase 1.

---

### M12. Monthly Statements — Phase 1

- **Purpose:** Automatically produce and deliver each farmer's previous-month statement.
- **Actors:** System; Farmer (recipient; portal viewer `[NEW]`).
- **Responsibilities:** Scheduled generation; PDF creation; WhatsApp delivery; status tracking.
- **Features:**
  - Generate on the 1st of each month for the previous month (`[SOURCE §13]`)
  - Fields: farmer ID + name, period, opening balance (if applicable), all purchase invoices (product, qty, rate, amount), all payments (dates + UTRs), closing/outstanding (`[SOURCE §13]`)
  - PDF generation (`[SOURCE §13]`)
  - WhatsApp delivery (`[SOURCE §13]`)
  - Statuses: generated/sent/delivered/failed/retry (`[SOURCE §13]`)
- **Inputs:** Ledger and payment data for the period.
- **Outputs:** Statement PDF; delivery status.
- **Business rules:**
  - Timing: 1st of month, previous-month window (`[SOURCE §13]`).
  - Exact time/timezone and retry policy — Not specified (OQ-14).
- **Dependencies:** M11 (ledger), M13 (WhatsApp), M14 (delivery status), M05 (portal access).
- **Permissions:** System scheduled; admin visibility.
- **Data involved:** Statement artifacts, delivery statuses.
- **Notifications:** The statement itself is delivered via WhatsApp.
- **Audit requirements:** Generation/delivery statuses recorded (`[SOURCE §13, §17]`).
- **Phase:** Phase 1.

---

### M13. WhatsApp Integration — Phase 1 (outbound) + Phase 2 (chatbot)

- **Purpose:** Deliver and (Phase 2) receive WhatsApp messages to/from farmers.
- **Actors:** System; WhatsApp provider; Farmer.
- **Responsibilities:** Send purchase/payment/statement messages; identify farmer by mobile; support chatbot in Phase 2.
- **Features:**
  - Purchase notification (`[SOURCE §8]`)
  - Payment notification with UTR (`[SOURCE §12]`)
  - Statement delivery (`[SOURCE §13]`)
  - Farmer identified by registered mobile linked to Farmer ID (`[SOURCE §3]`)
  - Phase 2: chatbot inbound handling (`[SOURCE §21]`)
- **Inputs:** Outbound message events; (Phase 2) inbound farmer queries.
- **Outputs:** WhatsApp messages; delivery status.
- **Business rules:**
  - Notification content as per `[SOURCE §8, §12]`; statement per `[SOURCE §13]`.
  - Provider and message policy — Not specified (OQ-04).
- **Dependencies:** M03 (mobile), M06/M08 (purchase trigger), M09/M10 (payment trigger), M12 (statement), M18 (integration settings), M14 (status tracking).
- **Permissions:** System integration (secure API auth `[SOURCE §18]`).
- **Data involved:** Message templates, delivery logs.
- **Notifications:** This module is the delivery channel.
- **Audit requirements:** Delivery statuses recorded (`[SOURCE §13]`); integration activity logged (`[SOURCE §17]`).
- **Phase:** Phase 1 (outbound); Phase 2 (chatbot).

---

### M14. Notifications — Phase 1

- **Purpose:** Coordinate notification creation, content and delivery status across channels (WhatsApp; portal in Phase 1 `[NEW]`).
- **Actors:** System; Farmer; Admin.
- **Responsibilities:** Trigger notifications on business events; track delivery status; expose history.
- **Features:**
  - Purchase notification trigger (`[SOURCE §8]`)
  - Payment notification trigger (`[SOURCE §12]`)
  - Statement delivery tracking incl. failed/retry (`[SOURCE §13]`)
  - Portal notification history (`[NEW]`)
- **Inputs:** Business events (invoice, payment, statement); delivery responses.
- **Outputs:** Notifications queue; delivery statuses; farmer-visible history.
- **Business rules:**
  - Statement delivery statuses mandatory (`[SOURCE §13]`).
  - Status tracking for purchase/payment messages — Not specified (Proposed Enhancement; OQ-04 related).
- **Dependencies:** M06/M08, M09/M10, M12, M13, M05.
- **Permissions:** System-managed; admin visibility.
- **Data involved:** Notification records, statuses, template references.
- **Notifications:** The notifications themselves.
- **Audit requirements:** Status transitions recorded (`[SOURCE §17]`).
- **Phase:** Phase 1.

---

### M15. Reports — Phase 1 (+ Phase 2 additions)

- **Purpose:** Provide standard business and analytical reports.
- **Actors:** Super Admin; Employee (per role).
- **Responsibilities:** Generate, filter, export reports.
- **Features:**
  - Farmer: farmer-wise purchase, payment, outstanding, ledger, monthly statement (`[SOURCE §14]`)
  - Procurement: date-wise, product-wise, employee-wise, farmer-wise, quantity-wise, rate-wise (`[SOURCE §14]`)
  - Payment: payment register, UTR register, paid, partially paid, pending, unreconciled (`[SOURCE §14]`)
  - Export: Excel, PDF, CSV where appropriate (`[SOURCE §14]`)
  - Phase 2: area-wise and employee performance reports (`[SOURCE §24]`, see M23)
- **Inputs:** Business data; date/filter parameters.
- **Outputs:** Report views/exports.
- **Business rules:**
  - Report definitions/filters/scheduling — Not specified in the source requirements (OQ related).
- **Dependencies:** M03, M06, M08, M09, M11, M12, M16, M20–M23 (Phase 2).
- **Permissions:** Admin full (`[SOURCE §2]`); others per permission matrix (`[SOURCE §16]`).
- **Data involved:** Aggregate/transaction data.
- **Notifications:** None specified.
- **Audit requirements:** Report queries only if required (Not specified in source).
- **Phase:** Phase 1 (+ Phase 2 additions).

---

### M16. Dashboard — Phase 1 (+ Phase 2 additions)

- **Purpose:** At-a-glance operational KPIs for Admin.
- **Actors:** Super Admin.
- **Responsibilities:** Surface KPIs with date filters.
- **Features:**
  - Total farmers, active farmers, total employees, today's procurement, today's procurement value, monthly procurement, pending farmer payments, payments made, total outstanding, number of invoices, unreconciled bank transactions (`[SOURCE §15]`)
  - Date filters (`[SOURCE §15]`)
  - Phase 2: area-wise performance views (`[SOURCE §23]`)
- **Inputs:** Aggregated operational data.
- **Outputs:** KPI dashboard.
- **Business rules:** KPI set fixed by `[SOURCE §15]`; refresh cadence not specified.
- **Dependencies:** M03, M04, M06, M08, M09, M11, M20 (Phase 2).
- **Permissions:** Admin full access (`[SOURCE §2]`).
- **Data involved:** Aggregates over farmers, employees, procurement, payments, invoices, reconciliation.
- **Notifications:** None.
- **Audit requirements:** View events not specified.
- **Phase:** Phase 1 (+ Phase 2 additions).

---

### M17. Audit Logs — Phase 1

- **Purpose:** Record every business action for accountability and financial integrity.
- **Actors:** System; Super Admin (view).
- **Responsibilities:** Capture; preserve; expose to admin.
- **Features:**
  - Trail fields: user/employee ID, date/time, action, original value, new value, record affected (`[SOURCE §17]`)
  - IP/device where appropriate (`[SOURCE §17]`)
  - Invoice modification tracking (`[SOURCE §6]`)
  - Admin review (`[SOURCE §2]`)
- **Inputs:** All business actions.
- **Outputs:** Append-only audit trail.
- **Business rules:**
  - No silent overwrite of historical financial records (`[SOURCE §17]`).
  - Retention/archival — Not specified (OQ-11).
- **Dependencies:** Every mutating module (M02–M14, M19–M22).
- **Permissions:** Admin view (`[SOURCE §2]`).
- **Data involved:** Audit records.
- **Notifications:** None.
- **Audit requirements:** This module is the audit mechanism.
- **Phase:** Phase 1.

---

### M18. System Settings — Phase 1

- **Purpose:** Configure the system: integrations, permissions, and operational settings.
- **Actors:** Super Admin.
- **Responsibilities:** Manage settings; WhatsApp and banking integrations; permission matrix.
- **Features:**
  - Manage settings, WhatsApp and banking integrations (`[SOURCE §2]`)
  - Configurable permission matrix (`[SOURCE §16]`)
- **Inputs:** Configuration values; integration credentials (secure).
- **Outputs:** Active configuration driving modules M10, M13, M02.
- **Business rules:**
  - Integration setup details — Not specified in the source requirements (OQ-03, OQ-04).
- **Dependencies:** M10, M13, M02.
- **Permissions:** Admin only (`[SOURCE §2]`).
- **Data involved:** Configuration, integration credentials (sensitive — `[SOURCE §18]`).
- **Notifications:** None.
- **Audit requirements:** Configuration changes recorded (`[SOURCE §17]`).
- **Phase:** Phase 1.

---

### M19. WhatsApp Chatbot — Phase 2

- **Purpose:** Allow farmers to query their own data via WhatsApp.
- **Actors:** Farmer; System; WhatsApp provider.
- **Responsibilities:** Answer farmer queries using the registered mobile → Farmer ID.
- **Features:**
  - Queries: My Outstanding, My Ledger, My Purchases, My Payments, Last Payment, Last Purchase, Invoice Details, Statement, Payment Status (`[SOURCE §21]`)
- **Inputs:** Inbound WhatsApp messages.
- **Outputs:** Text/statement responses.
- **Business rules:** Farmer identified by registered mobile (`[SOURCE §3, §21]`).
- **Dependencies:** M13 (channel), M03 (identity), M11 (ledger), M08 (invoices), M09 (payments), M12 (statements).
- **Permissions:** Farmer — own data only.
- **Data involved:** Farmer's own periodic data.
- **Notifications:** Responses to queries.
- **Audit requirements:** Query/response logging recommended (Not specified; Proposed Enhancement).
- **Phase:** Phase 2.

---

### M20. Area Management — Phase 2

- **Purpose:** Define operational/geographical areas.
- **Actors:** Super Admin.
- **Responsibilities:** Create and manage areas.
- **Features:**
  - Create/manage areas (`[SOURCE §23]`)
- **Inputs:** Area definitions.
- **Outputs:** Area catalogue.
- **Business rules:** Area modelling attributes — Not specified in the source requirements.
- **Dependencies:** M02, M03, M04, M21, M22.
- **Permissions:** Admin full access (`[SOURCE §2]` maintained in Phase 2 `[SOURCE §23]`).
- **Data involved:** Area master.
- **Notifications:** None.
- **Audit requirements:** Area changes recorded (`[SOURCE §17]`).
- **Phase:** Phase 2.

---

### M21. Area-based Farmer Allocation — Phase 2

- **Purpose:** Assign farmers to areas and restrict employee exposure to assigned-area farmers.
- **Actors:** Super Admin; System (authorization).
- **Responsibilities:** Assign farmers to areas; reassign; enforce employee area restriction.
- **Features:**
  - Assign farmers to areas (`[SOURCE §23]`)
  - Reassign farmers (`[SOURCE §23]`)
  - Restrict employee view to assigned area (`[SOURCE §22]`)
- **Inputs:** Farmer ↔ area assignments.
- **Outputs:** Authorized farmer data per area.
- **Business rules:**
  - Employees cannot search/view/edit/download farmers outside their area (`[SOURCE §22]`)
  - Enforcement at backend/database authorization level (`[SOURCE §22]`)
- **Dependencies:** M03, M20, M22.
- **Permissions:** Admin assignment; employee restricted by area.
- **Data involved:** Farmer-area assignments.
- **Notifications:** None specified.
- **Audit requirements:** Assignments/reassignments recorded (`[SOURCE §17]`).
- **Phase:** Phase 2.

---

### M22. Area-based Employee Allocation — Phase 2

- **Purpose:** Assign employees to areas so area-wise authorization applies.
- **Actors:** Super Admin.
- **Responsibilities:** Assign employees to areas; transfer employees.
- **Features:**
  - Assign employees to areas (`[SOURCE §4, §23]`; area field reserved in Phase 1)
  - Transfer employees (`[SOURCE §23]`)
- **Inputs:** Employee ↔ area assignments.
- **Outputs:** Employee area scopes.
- **Business rules:** Area field reserved since Phase 1 (`[SOURCE §4]`); employee restricted to own area (`[SOURCE §22]`).
- **Dependencies:** M04, M20, M21.
- **Permissions:** Admin access.
- **Data involved:** Employee-area assignments.
- **Notifications:** None specified.
- **Audit requirements:** Assignments recorded (`[SOURCE §17]`).
- **Phase:** Phase 2.

---

### M23. Area-wise Reporting — Phase 2

- **Purpose:** Report and monitor operations by area and by employee.
- **Actors:** Super Admin / Management.
- **Responsibilities:** Area-wise and employee-performance reporting.
- **Features:**
  - Area-wise farmer count, procurement, payment, outstanding (`[SOURCE §24]`)
  - Employee-wise: farmers handled, invoices created, quantity purchased, purchase value, average rate (`[SOURCE §24]`)
  - Corrections and cancellations (`[SOURCE §24]`)
  - Area-wise performance views (`[SOURCE §23]`)
- **Inputs:** Area, farmer, procurement, payment, invoice data.
- **Outputs:** Area/employee reports.
- **Business rules:** No additional rules specified.
- **Dependencies:** M15, M20, M21, M22.
- **Permissions:** Admin/Management (`[SOURCE §23]`).
- **Data involved:** Aggregates by area/employee.
- **Notifications:** None.
- **Audit requirements:** As per general reporting.
- **Phase:** Phase 2.

---

## 4. Future / Proposed Modules

| # | Capability | Status | Basis |
|---|---|---|---|
| F-01 | Future-ready operational fields: Area (named), Collection centre, GPS, Quality/grade, Lot number, Batch number, Warehouse, Packing, Export shipment | Future — inactive in Phase 1, architected | `[SOURCE §20]` |
| F-02 | Product catalogue & rate governance (rate approval workflows) | Proposed Enhancement | OQ-05 |
| F-03 | Over-allocation validation for payments | Proposed Enhancement | Associated OQ/BRD §22 |
| F-04 | Farmer portal support/ticketing | Proposed (channel undefined) | OQ-16 |
| F-05 | Notification delivery tracking for purchase/payment messages | Proposed Enhancement | OQ-04 related |
| F-06 | Manual re-send of failed WhatsApp messages by admin | Proposed Enhancement | BRD W23 |
| F-07 | Farmer portal account activation & onboarding flow | Required for portal; procedure undefined | OQ-15 |

---

## 5. Cross-Module Dependencies (summary)

Core data chain: M03 (Farmer) → M06 (Procurement) → M08 (Invoice) → M11 (Ledger) → M09/M10 (Payment) → M12 (Statement) → M13/M14 (WhatsApp).

Supporting modules: M01/M02/M04/M18 (identity, users, settings) and M17 (audit) serve all modules. M05 (Farmer Portal) and M19 (chatbot, Phase 2) are farmer-facing consumers of the same data; M20–M23 add area dimensions in Phase 2.

---

*End of Module Specification v1.0. Next in sequence: `05_DATA_MODEL.md` (per index reading order).*