# Agri Procurement & Farmer Management System
## User Roles and User Stories

---

## 1. Document Information

| Item | Value |
|---|---|
| Product name | Agri Procurement & Farmer Management System |
| Document | User Roles and User Stories |
| Version | v1.0 |
| Status | Draft — aligned to PRD v1.0, BRD v1.0, Business Workflows v1.0, Module Specification v1.0 |
| Date | 2026-09-16 |
| Purpose | Define the three system roles (permissions, modules, workflows) and an acceptance-ready set of user stories by feature area |
| Source | 1. `AgriProcurement & Farmer Management.pdf` (primary source of truth) 2. `docs/00_DOCUMENTATION_INDEX.md` 3. `docs/01_PRODUCT_REQUIREMENTS_DOCUMENT.md` 4. `docs/02_BUSINESS_REQUIREMENTS_DOCUMENT.md` 5. `docs/03_BUSINESS_WORKFLOWS.md` 6. `docs/04_MODULE_SPECIFICATION.md` |

### Conventions

| Tag | Meaning |
|---|---|
| `[SOURCE §n]` | Requirement from the original PDF, section `n` |
| `[NEW]` | Newly approved Farmer Portal requirement — not in the original PDF |
| Module ref | `M01`…`M23` refer to `docs/04_MODULE_SPECIFICATION.md` |
| Undefined | Not described in the source; tracked in PRD §28 / BRD §25 |

---

## 2. Roles

### 2.1 Super Admin

| Attribute | Detail |
|---|---|
| Role purpose | Run the entire operation from one place with full control and visibility (`[SOURCE §2]`). |
| Responsibilities | Manage employees and farmers; view all areas, procurement, invoices, payments, ledgers and reports; manage settings, WhatsApp and banking integrations; view audit logs; configure the permission matrix (`[SOURCE §2, §16]`). |
| Permissions | Full access to the entire system (`[SOURCE §2]`); configures permissions for all other roles (`[SOURCE §16]`). |
| Accessible modules | M01–M18 (Phase 1); in Phase 2 additionally M19–M23 (area management, allocation, area reporting) with full Admin/Management access maintained (`[SOURCE §23]`). |
| Restricted modules | None within scope. |
| Typical workflows | Admin login (BRD W07); report generation/exports (W29); audit review (W30); master data management; integration & settings configuration; permission matrix management. |

### 2.2 Procurement Employee

| Attribute | Detail |
|---|---|
| Role purpose | Record purchases and generate invoices while collecting material directly from farmers (`[SOURCE §2, §5]`). |
| Responsibilities | Individual login; identify farmer; capture product, weight, rate; confirm transaction; see generated invoice; every transaction is attributed to this employee (backend trail) (`[SOURCE §2, §5]`). |
| Permissions | Limited to farmer information required for procurement and procurement entry (`[SOURCE §16]`). Initial target 100 employees; architecture supports 500+ (`[SOURCE §2, §19]`). |
| Accessible modules | M01 (authentication); farmer lookup within M03 (restricted scope per `[SOURCE §16]`); M06 (procurement); invoice confirmation via M08. |
| Restricted modules | M02 (user admin), M04 (employee admin), M09/M10 (payments & reconciliation), M16 (Admin Dashboard), M17 (audit logs), M18 (system settings), M12 (statements). Access to other modules, if any, depends on the Admin-configured permission matrix (`[SOURCE §16]`). |
| Typical workflows | Employee login (BRD W06); procurement entry (W08); invoice confirmation (W09). |

### 2.3 Farmer

| Attribute | Detail |
|---|---|
| Role purpose | Sell produce and stay informed of purchases, payments and statements; access own records (`[NEW]` portal; WhatsApp per source). |
| Responsibilities | Supply produce under their Farmer ID; receive WhatsApp notifications; (Phase 2) query own data via WhatsApp chatbot (`[SOURCE §21]`); (`[NEW]`) log in to the portal and view own data. |
| Permissions | Own-data only, enforced at backend level (`[NEW]`; derived from `[SOURCE §22]` access philosophy). No native mobile app in Phase 1 (`[SOURCE §1]`; `[NEW]` N-03). |
| Accessible modules | M05 Farmer Portal (`[NEW]`); WhatsApp messages (via M13/M14); Phase 2 M19 (chatbot). Read-only own views of M08/M09/M11/M12 data. |
| Restricted modules | All management modules (M02, M03, M04, M06, M07, M16, M17, M18, M20–M23). Farmers cannot view other farmers' data. |
| Typical workflows | Farmer login and auth (BRD W03/W04 — OTP is Proposed Enhancement); portal self-service (W24–W28); receives purchase/payment/statement notifications. |

---

## 3. User Stories

### 3.1 Authentication

**US-001 — Employee login**
Role: Procurement Employee
As a Procurement Employee, I want to log in with my individual credentials, so that I can enter purchases and generate invoices under my own identity.
- Acceptance Criteria:
  - Given that I am a registered employee with valid credentials,
  - When I log in to the portal,
  - Then I am authenticated and granted procurement access per my role (`[SOURCE §2, §16]`), and my session is managed securely (`[SOURCE §18]`).

**US-002 — Admin login**
Role: Super Admin
As a Super Admin, I want to log in with full system access, so that I can manage the entire operation.
- Acceptance Criteria:
  - Given that I am a Super Admin with valid credentials,
  - When I log in,
  - Then I receive full access to the entire system (`[SOURCE §2]`).

**US-003 — Secure authentication controls**
Role: Any user / System
As a user, I want secure authentication with strong password policy and OTP/2FA where appropriate, so that my account is protected.
- Acceptance Criteria:
  - Given the security baseline,
  - When authentication is attempted,
  - Then the system enforces secure authentication, strong password policy, OTP/2FA where appropriate, secure API authentication, session management, and rate limiting (`[SOURCE §18]`).

**US-004 — Role-based authorization**
Role: Any user / System
As a user, I want access enforced by role-based authorization, so that I only see what my role permits.
- Acceptance Criteria:
  - Given a configured permission matrix (`[SOURCE §16]`),
  - When a user attempts an action,
  - Then the system grants access only as permitted by the user's role (`[SOURCE §18]`).

### 3.2 Farmer Management

**US-005 — Register a farmer**
Role: Super Admin
As a Super Admin, I want to register farmers with their master details, so that every farmer has a trusted record in the system.
- Acceptance Criteria:
  - Given that I am an authenticated admin,
  - When I create a farmer,
  - Then the system records farmer name, mobile, address, village, taluka, district, state, bank name, account number, IFSC, KYC where required, registration date, status, products normally supplied, and internal remarks (`[SOURCE §3]`).

**US-006 — Issue permanent Farmer ID**
Role: Super Admin / System
As a Super Admin, I want each farmer to receive a permanent unique Farmer ID (e.g., F-0001), so that the farmer has a permanent primary business identity.
- Acceptance Criteria:
  - Given that a farmer record is being created,
  - When the ID is assigned,
  - Then the Farmer ID is permanent, unique, and is the farmer's primary business identity (`[SOURCE §3]`).
  - And two farmers can never share the same ID.

**US-007 — Link mobile for WhatsApp**
Role: Super Admin / System
As a Super Admin, I want the registered mobile number linked to the Farmer ID, so that WhatsApp identification of the farmer works.
- Acceptance Criteria:
  - Given a farmer with a registered mobile number,
  - When the farmer master is saved,
  - Then the mobile number is linked to the Farmer ID and is usable as the WhatsApp identity (`[SOURCE §3]`).

**US-008 — Employee farmer lookup**
Role: Procurement Employee
As a Procurement Employee, I want to search/select farmers, so that I can start a purchase entry quickly.
- Acceptance Criteria:
  - Given that I am on the procurement screen (`[SOURCE §5]`),
  - When I search or select a farmer,
  - Then I see the farmer information required for procurement only (`[SOURCE §16]`).
  - And I cannot access farmers or data outside my permitted scope (Phase 2: my assigned area) (`[SOURCE §22]`).

### 3.3 Employee Management

**US-009 — Create an employee**
Role: Super Admin
As a Super Admin, I want to create employee records, so that employees can log in and operate.
- Acceptance Criteria:
  - Given that I am an admin,
  - When I create an employee,
  - Then the record captures Employee ID (e.g., EMP-001), name, mobile number, email, authentication credentials, role and status (`[SOURCE §4]`).

**US-010 — Reserve area field**
Role: Super Admin / System
As a Super Admin, I want the area/location field reserved on employee records, so that Phase 2 area allocation can be enabled without redesign.
- Acceptance Criteria:
  - Given the employee data model,
  - When configured,
  - Then the area/location field exists but remains inactive in Phase 1 (`[SOURCE §4, §20]`) and becomes active for area assignment in Phase 2 (`[SOURCE §23]`).

**US-011 — Employee scalability**
Role: Super Admin / System
As a Super Admin, I want support for 100 employees initially with architecture for 500, so that the business can grow logins without redesign.
- Acceptance Criteria:
  - Given the target employee count,
  - When usage grows,
  - Then the architecture supports at least 500 employees and concurrent use (`[SOURCE §2, §19]`).

### 3.4 Procurement

**US-012 — Capture a purchase**
Role: Procurement Employee
As a Procurement Employee, I want to follow the guided purchase workflow, so that purchases are recorded correctly in the field.
- Acceptance Criteria:
  - Given that I am logged in,
  - When I select/search the farmer, select the product, enter weight and rate,
  - Then the system follows: Login → Select/Search Farmer → Select Product → Enter Weight → Enter Rate → Calculate Amount → Confirm → Generate Invoice (`[SOURCE §5]`).

**US-013 — Automatic amount calculation**
Role: Procurement Employee / System
As a Procurement Employee, I want the amount calculated automatically, so that calculation errors are avoided.
- Acceptance Criteria:
  - Given quantity and rate are entered,
  - When I confirm,
  - Then the amount equals quantity × rate, calculated automatically (`[SOURCE §5]`).

**US-014 — Deduction and net amount**
Role: Procurement Employee / System
As a Procurement Employee, I want to record a deduction when applicable, so that the net amount reflects the final payable.
- Acceptance Criteria:
  - Given a deduction applies,
  - When the transaction is entered,
  - Then gross amount, deduction and net amount are recorded (`[SOURCE §5]`).
  - Note: deduction types and approval rules are undefined (OQ-06).

**US-015 — Transaction attribution**
Role: System
As a System, I want every transaction to record the creating employee, so that every action has a backend trail.
- Acceptance Criteria:
  - Given any purchase transaction,
  - When it is saved,
  - Then the employee who created it is recorded (backend trail) (`[SOURCE §2]`).

### 3.5 Invoice

**US-016 — Automatic invoice numbering**
Role: System
As a System, I want invoice numbers generated from the Farmer ID and transaction sequence, so that numbering is always correct and unique.
- Acceptance Criteria:
  - Given a confirmed transaction (`[SOURCE §5]`),
  - When the invoice is generated,
  - Then the number is formed from Farmer ID + per-farmer sequence (e.g., F-0001-17) (`[SOURCE §6]`).
  - And Farmer ID and transaction sequence are stored as separate fields (`[SOURCE §6]`).
  - And the number is unique with an independent per-farmer sequence (`[SOURCE §6]`).

**US-017 — No manual invoice numbering**
Role: Procurement Employee / System
As a System, I want invoice numbers to never be typed manually, so that numbering integrity is preserved.
- Acceptance Criteria:
  - Given any employee action,
  - When an invoice is generated,
  - Then the system, not the employee, assigns the number (`[SOURCE §6]`).

**US-018 — Cancellation without reuse**
Role: Authorised user (actor undefined, OQ-08) / System
As a System, I want cancelled invoices retained with their numbers never reused, so that the invoice history stays accurate.
- Acceptance Criteria:
  - Given an invoice is cancelled,
  - When cancellation is recorded,
  - Then the invoice remains in the system and its number is never reused (`[SOURCE §6]`).

**US-019 — Invoice confirmation display**
Role: Procurement Employee
As a Procurement Employee, I want to see the transaction confirmation, so that I can verify what was recorded.
- Acceptance Criteria:
  - Given an invoice is generated,
  - When I view the confirmation,
  - Then it shows Farmer, Invoice, Date, Product, Quantity, Rate, Total and Employee (`[SOURCE §7]`).

**US-020 — Invoice change audit**
Role: System
As a System, I want every invoice modification recorded in the audit log, so that changes are traceable.
- Acceptance Criteria:
  - Given any invoice modification,
  - When it occurs,
  - Then it is written to the audit log (`[SOURCE §6, §17]`).

### 3.6 Payment

**US-021 — Record a payment**
Role: Payment creator (actor undefined; see note) 
As an authorised user, I want to record payments with full detail, so that payments are traceable to invoices.
- Acceptance Criteria:
  - Given a payment is due,
  - When I record it,
  - Then the system captures Farmer ID, invoice number/allocation, payment amount, payment date, payment status, payment mode, bank reference, UTR and remarks (`[SOURCE §10]`).
  - Note: the creating role is undefined; reconciliation queue is assigned to accounts staff (`[SOURCE §11]`).

**US-022 — Full payment**
Role: Authorised user
As an authorised user, I want to settle an invoice fully, so that the invoice is marked paid.
- Acceptance Criteria:
  - Given an invoice with total due,
  - When a full payment is allocated,
  - Then the invoice becomes fully paid (`[SOURCE §10]`).

**US-023 — Partial payment**
Role: Authorised user
As an authorised user, I want to accept a partial payment, so that a farmer can pay in part.
- Acceptance Criteria:
  - Given an invoice,
  - When a partial payment is recorded,
  - Then the invoice remains partially outstanding (`[SOURCE §10]`) and appears in "partially paid" reporting (`[SOURCE §14]`).

**US-024 — Multiple payments against an invoice**
Role: Authorised user
As an authorised user, I want multiple payments against one invoice, so that an invoice can be settled over instalments.
- Acceptance Criteria:
  - Given an invoice,
  - When several payments are recorded against it,
  - Then each payment is tracked and the invoice is settled once cumulative payments reach the total (`[SOURCE §10]`).

**US-025 — One payment against multiple invoices**
Role: Authorised user
As an authorised user, I want one payment allocated to several invoices, so that a single payout can clear multiple dues.
- Acceptance Criteria:
  - Given a single payout,
  - When it is allocated across multiple invoices,
  - Then each invoice's outstanding reduces by its allocation (`[SOURCE §10]`).

### 3.7 Ledger

**US-026 — Farmer-wise ledger**
Role: Super Admin / System
As a Super Admin, I want a complete farmer-wise ledger, so that every farmer's financial position is visible.
- Acceptance Criteria:
  - Given farmer transactions,
  - When invoices and payments are recorded,
  - Then the system maintains a complete farmer-wise ledger (`[SOURCE §9]`), showing date, invoice, product, quantity, rate, amount, payment and UTR (`[SOURCE §9]`).

**US-027 — Automatic ledger update on confirmed payment**
Role: System
As a System, I want the ledger updated automatically when a payment is confirmed, so that the ledger is always current.
- Acceptance Criteria:
  - Given a bank/API-confirmed payment,
  - When confirmation is received,
  - Then the farmer ledger is updated automatically (`[SOURCE §11]`).

**US-028 — No silent overwrite**
Role: System
As a System, I want historical financial records preserved, so that past records are never silently changed.
- Acceptance Criteria:
  - Given any correction to financial data,
  - When a change is made,
  - Then historical records are not silently overwritten; changes are captured through the audit process (`[SOURCE §17]`).

### 3.8 Statements

**US-029 — Monthly statement generation**
Role: System
As a System, I want to generate the previous month's statement automatically on the 1st, so that every farmer receives a timely statement.
- Acceptance Criteria:
  - Given the start of a new month,
  - When the 1st day arrives,
  - Then the system generates the previous month's statement (e.g., on 1 September it generates 1–31 August) (`[SOURCE §13]`).

**US-030 — Statement content**
Role: System
As a System, I want complete statement content, so that farmers see a full picture of the month.
- Acceptance Criteria:
  - Given a generated statement,
  - When it is produced,
  - Then it includes Farmer ID and name, statement period, opening balance (if applicable), all purchase invoices (product, quantity, rate, amount), all payments (dates and UTRs), and closing/outstanding balance (`[SOURCE §13]`).

**US-031 — Statement PDF**
Role: System
As a System, I want the statement as a PDF, so that it can be delivered and printed reliably.
- Acceptance Criteria:
  - Given statement content,
  - When the statement is generated,
  - Then a PDF is produced (`[SOURCE §13]`).

**US-032 — Statement delivery and status**
Role: System
As a System, I want statement delivery tracked, so that failures are visible and retryable.
- Acceptance Criteria:
  - Given a statement is sent via WhatsApp,
  - When delivery is attempted,
  - Then the status is recorded as generated, sent, delivered, failed or retry (`[SOURCE §13]`).

### 3.9 WhatsApp

**US-033 — Purchase notification**
Role: System / Farmer
As a Farmer, I want to be notified on WhatsApp after my purchase is invoiced, so that I know the transaction is recorded.
- Acceptance Criteria:
  - Given a successful invoice creation (`[SOURCE §5]`),
  - When the invoice is generated,
  - Then an automatic WhatsApp purchase notification is sent to my registered number (`[SOURCE §8]`) carrying the purchase confirmation details.

**US-034 — Payment notification with UTR**
Role: System / Farmer
As a Farmer, I want to be notified when my payment is credited, so that I have the confirmation and UTR.
- Acceptance Criteria:
  - Given a payment confirmed by the bank/API (`[SOURCE §11]`),
  - When confirmation is received,
  - Then an automatic WhatsApp payment notification with the amount and UTR is sent to the farmer (`[SOURCE §12]`).

**US-035 — Farmer identity on WhatsApp**
Role: System
As a System, I want to identify farmers by their registered mobile, so that messages reach the right farmer.
- Acceptance Criteria:
  - Given a mobile number linked to a Farmer ID,
  - When a WhatsApp message is sent or a query arrives,
  - Then the farmer is identified by the registered mobile number tied to the Farmer ID (`[SOURCE §3, §21]`).

### 3.10 Reports

**US-036 — Farmer reports**
Role: Super Admin
As a Super Admin, I want farmer-wise reports, so that I can review purchases, payments, outstanding, ledger and statements per farmer.
- Acceptance Criteria:
  - Given farmer data,
  - When I generate a farmer report,
  - Then it supports farmer-wise purchase, payment, outstanding, ledger and monthly statement (`[SOURCE §14]`).

**US-037 — Procurement reports**
Role: Super Admin
As a Super Admin, I want procurement reports sliced in multiple ways, so that I can analyse buying activity.
- Acceptance Criteria:
  - Given procurement data,
  - When I generate a procurement report,
  - Then it supports date-wise, product-wise, employee-wise, farmer-wise, quantity-wise and rate-wise views (`[SOURCE §14]`).

**US-038 — Payment reports**
Role: Super Admin
As a Super Admin, I want payment reports, so that payments are fully visible and reconcilable.
- Acceptance Criteria:
  - Given payment data,
  - When I generate a payment report,
  - Then it includes the payment register, UTR register, paid, partially paid, pending and unreconciled payments (`[SOURCE §14]`).

**US-039 — Report export**
Role: Super Admin
As a Super Admin, I want reports in Excel, PDF and CSV, so that they can be shared and analysed.
- Acceptance Criteria:
  - Given a report,
  - When I export it,
  - Then it is available in Excel, PDF and CSV where appropriate (`[SOURCE §14]`).

**US-040 — Admin dashboard KPIs**
Role: Super Admin
As a Super Admin, I want a dashboard of key figures, so that I can monitor the operation at a glance.
- Acceptance Criteria:
  - Given operational data,
  - When I open the Admin Dashboard,
  - Then it shows total farmers, active farmers, total employees, today's procurement, today's procurement value, monthly procurement, pending farmer payments, payments made, total outstanding, number of invoices, and unreconciled bank transactions (`[SOURCE §15]`).
  - And I can apply date filters (`[SOURCE §15]`).

### 3.11 Audit

**US-041 — Audit trail capture**
Role: System
As a System, I want full audit records, so that every action is attributable.
- Acceptance Criteria:
  - Given any business action,
  - When it is recorded,
  - Then the audit entry contains user/employee ID, date and time, action, original value, new value and record affected, with IP/device where appropriate (`[SOURCE §17]`).

**US-042 — Admin audit review**
Role: Super Admin
As a Super Admin, I want to view audit logs, so that I can investigate any action in the system.
- Acceptance Criteria:
  - Given audit records exist,
  - When I access audit logs,
  - Then I can review them (admin-only) (`[SOURCE §2]`).

### 3.12 Farmer Portal — `[NEW]` newly approved requirements

> All stories below are **newly approved requirements (`[NEW]`)** , not part of the original PDF. Detailed behaviour of some modules is undefined (OQ-01); where relevant this is noted.

**US-043 — Farmer login**
Role: Farmer
As a Farmer, I want to log in to a Farmer Portal, so that I can access my own records myself.
- Acceptance Criteria:
  - Given that I am a registered farmer,
  - When I open the portal in a web/mobile browser and log in,
  - Then I am authenticated and can access my own data (`[NEW]`, N-01).
  - And the portal is responsive; there is no native mobile application in Phase 1 (`[NEW]` N-03; `[SOURCE §1]`).
  - Note: authentication method (e.g., OTP) is a Proposed Enhancement pending OQ-02.

**US-044 — Farmer dashboard**
Role: Farmer
As a Farmer, I want a personal dashboard, so that I quickly see my own position.
- Acceptance Criteria:
  - Given I am logged in,
  - When I open the dashboard,
  - Then I see a summary of my own purchases, payments and outstanding (`[NEW]`, N-02).
  - Note: exact widgets are undefined (OQ-01).

**US-045 — My Profile**
Role: Farmer
As a Farmer, I want to view my profile, so that I can confirm my registered details.
- Acceptance Criteria:
  - Given I am logged in,
  - When I open My Profile,
  - Then I can view my own master data (identity, contact, village/taluka/district/state, bank, KYC status) (`[NEW]`).
  - Note: editing scope for profile fields is undefined (OQ-18).

**US-046 — My Purchases**
Role: Farmer
As a Farmer, I want to see my purchases, so that I can confirm what has been bought from me.
- Acceptance Criteria:
  - Given I am logged in,
  - When I open My Purchases,
  - Then I see a list of my own purchase transactions (`[NEW]`).

**US-047 — My Invoices**
Role: Farmer
As a Farmer, I want to see my invoices, so that I can track my invoiced sales.
- Acceptance Criteria:
  - Given I am logged in,
  - When I open My Invoices,
  - Then I see my own invoices and their status (`[NEW]`).

**US-048 — My Payments**
Role: Farmer
As a Farmer, I want to see my payments, so that I can check what has been paid to me.
- Acceptance Criteria:
  - Given I am logged in,
  - When I open My Payments,
  - Then I see my own payments including date, mode, UTR and status (`[NEW]`).

**US-049 — My Ledger**
Role: Farmer
As a Farmer, I want to see my ledger, so that I can verify my standing balance.
- Acceptance Criteria:
  - Given I am logged in,
  - When I open My Ledger,
  - Then I see my own ledger (per `[SOURCE §9]` content) and outstanding (`[NEW]`).

**US-050 — My Statements**
Role: Farmer
As a Farmer, I want to view my monthly statements, so that I can access them anytime.
- Acceptance Criteria:
  - Given I am logged in,
  - When I open My Statements,
  - Then I can view/download my own monthly statement PDFs (`[NEW]`; statement content per `[SOURCE §13]`).

**US-051 — Notifications**
Role: Farmer
As a Farmer, I want a notifications view, so that I can revisit messages I have received.
- Acceptance Criteria:
  - Given I am logged in,
  - When I open Notifications,
  - Then I see my own notification history (purchase, payment, statement) (`[NEW]`).

**US-052 — Support**
Role: Farmer
As a Farmer, I want a support entry point, so that I can get help with my account.
- Acceptance Criteria:
  - Given I am logged in,
  - When I open Support,
  - Then I can find/use the defined contact channel (`[NEW]`).
  - Note: the support channel itself is undefined (OQ-16).

---

## 4. Requirement Coverage Notes

- All stories trace to source sections of the original PDF (`[SOURCE §n]`) or to newly approved Farmer Portal requirements (`[NEW]`).
- Where a behaviour is not defined in the source, the story notes it explicitly (e.g., US-014 deductions OQ-06, US-018 cancellation actor OQ-08, US-021 payment creator OQ, US-043 auth method OQ-02).
- No unsupported business rules were introduced.

---

*End of User Roles and User Stories v1.0. Next in sequence: `06_INTEGRATIONS.md` (or per index reading order).*