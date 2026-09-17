# Agri Procurement & Farmer Management System
## Functional Requirements Specification (FRS)

---

## 1. Document Information

| Item | Value |
|---|---|
| Product name | Agri Procurement & Farmer Management System |
| Document | Functional Requirements Specification (FRS) |
| Version | v1.0 |
| Status | Draft — aligned to PRD v1.0, BRD v1.0, Workflows v1.0, Modules v1.0, User Stories v1.0 |
| Date | 2026-09-16 |
| Purpose | Complete functional specification by module: requirement, description, actor, preconditions, inputs, processing/business logic, outputs, validation, error handling, permissions, audit, priority, phase, source |
| Source | 1. `AgriProcurement & Farmer Management.pdf` (primary source of truth — must not be contradicted) 2. `docs/00_DOCUMENTATION_INDEX.md` 3. `docs/01_PRODUCT_REQUIREMENTS_DOCUMENT.md` 4. `docs/02_BUSINESS_REQUIREMENTS_DOCUMENT.md` 5. `docs/03_BUSINESS_WORKFLOWS.md` 6. `docs/04_MODULE_SPECIFICATION.md` 7. `docs/05_USER_ROLES_AND_USER_STORIES.md` |

### Conventions

| Tag | Meaning |
|---|---|
| `[SOURCE §n]` | Requirement from original PDF section `n` |
| `[NEW]` | Newly approved Farmer Portal requirement |
| Priority | MUST / SHOULD / MAY |
| Phase | 1 (core), 2 (chatbot & areas) |
| Undefined | Not specified in the source requirements (see PRD §28 / BRD §25) |

No requirement below contradicts the source PDF. Undefined behaviour is explicitly marked, never invented.

---

## 2. Functional Requirements by Module

---

## 2.1 Authentication

### FR-AUTH-001 — Secure login for portal users
- **Requirement:** The portal MUST provide secure authentication for all users.
- **Description:** Identifies employees, admins and (NEW) farmers before allowing access to the portal.
- **Actor:** Super Admin; Procurement Employee; Farmer.
- **Preconditions:** User account exists; user has credentials.
- **Inputs:** Username/ID and credentials (password, and OTP/2FA where applicable).
- **Processing/business logic:** System verifies credentials and grants an authenticated session scoped to the user's role.
- **Outputs:** Authenticated session / access decision.
- **Validation:** Credentials must match the user record; failed attempts shall be limited.
- **Error handling:** Invalid credentials → access denied with safe error; retry governed by rate limiting.
- **Permissions:** By role after authentication (`[SOURCE §18]`).
- **Audit requirement:** Authentication events recorded (audit + session management; `[SOURCE §17, §18]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §18]`.

### FR-AUTH-002 — Strong password policy
- **Requirement:** The system MUST enforce a strong password policy.
- **Description:** Passwords meet defined complexity rules for admin/employee accounts.
- **Actor:** System; user.
- **Preconditions:** Password is being created or changed.
- **Inputs:** New password.
- **Processing/business logic:** Policy checks (length/complexity) per `[SOURCE §18]`; exact policy parameters not specified.
- **Outputs:** Accepted/rejected password.
- **Validation:** Must satisfy the configured policy.
- **Error handling:** Policy violation → reject with guidance.
- **Permissions:** User self-service or admin reset per permission matrix.
- **Audit requirement:** Password changes logged (`[SOURCE §17]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §18]`.

### FR-AUTH-003 — OTP / 2FA
- **Requirement:** The system MUST support OTP/2FA where appropriate.
- **Description:** Adds a second factor for sensitive operations/logins where the business deems it appropriate.
- **Actor:** System; user.
- **Preconditions:** User enabled for 2FA; factor available.
- **Inputs:** OTP/second factor value.
- **Processing/business logic:** Verify second factor after primary authentication.
- **Outputs:** Completed/incomplete authentication.
- **Validation:** Second factor must match issued token.
- **Error handling:** Mismatch → deny; resend governed by controls.
- **Permissions:** Per user/role policy.
- **Audit requirement:** 2FA events logged (`[SOURCE §17, §18]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §18]`.

### FR-AUTH-004 — Role-based authorization (RBAC)
- **Requirement:** The system MUST implement role-based authorization.
- **Description:** Access to features/data is controlled by role; permission matrix configurable by Admin.
- **Actor:** System; Super Admin.
- **Preconditions:** Roles and permission matrix configured.
- **Inputs:** Requested action; user role.
- **Processing/business logic:** Evaluate permission matrix against role and target feature/data.
- **Outputs:** Allow/deny.
- **Validation:** Matrix entries as configured by Admin (`[SOURCE §16]`).
- **Error handling:** Denied action → friendly denial/audit entry.
- **Permissions:** Admin full; others by matrix.
- **Audit requirement:** Denied attempts and authorization changes logged (`[SOURCE §17]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §18, §16]`.

### FR-AUTH-005 — Session management
- **Requirement:** The system MUST manage user sessions.
- **Description:** Sessions expire, can be terminated, and are secured.
- **Actor:** System; user.
- **Preconditions:** User authenticated.
- **Inputs:** Session operations.
- **Processing/business logic:** Maintain session lifecycle with secure handling.
- **Outputs:** Valid/expired/terminated session.
- **Validation:** Session validity enforced.
- **Error handling:** Expired/invalid session → re-authentication required.
- **Permissions:** User's own session; admin control where applicable.
- **Audit requirement:** Session events logged (`[SOURCE §17, §18]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §18]`.

### FR-AUTH-006 — Rate limiting and attack protection
- **Requirement:** The system MUST apply rate limiting and protect against common web/API attacks.
- **Description:** Defends authentication and API endpoints against abuse.
- **Actor:** System.
- **Preconditions:** Traffic arrives at portal/API.
- **Inputs:** Requests.
- **Processing/business logic:** Apply rate limits and attack protections (`[SOURCE §18]`).
- **Outputs:** Served/rejected requests.
- **Validation:** Per configured thresholds.
- **Error handling:** Threshold exceeded → throttle/block with logging.
- **Permissions:** System-level.
- **Audit requirement:** Abuse events logged (`[SOURCE §17]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §18]`.

### FR-AUTH-007 — Farmer portal authentication
- **Requirement:** The Farmer Portal MUST provide farmer login (`[NEW]`).
- **Description:** Farmers authenticate to access the portal. Method (e.g., OTP on registered mobile) is a Proposed Enhancement — final method pending (OQ-02).
- **Actor:** Farmer; System.
- **Preconditions:** Farmer's portal account exists.
- **Inputs:** Farmer credentials/OTP (per adopted method).
- **Processing/business logic:** Authenticate farmer; restrict to own data.
- **Outputs:** Authenticated farmer session.
- **Validation:** Per adopted method (undefined).
- **Error handling:** Invalid/missing factor → deny; resend governed by controls.
- **Permissions:** Farmer — own data only.
- **Audit requirement:** Login events logged (Proposed Enhancement consistent with `[SOURCE §18]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[NEW]` (N-01/N-02); OTP = Proposed Enhancement.

---

## 2.2 Farmer

### FR-FRM-001 — Farmer master creation
- **Requirement:** The system MUST allow creation of Farmer Master records with the defined fields.
- **Description:** Captures farmer name, mobile, address, village, taluka, district, state, bank name, account number, IFSC, KYC (where required), registration date, status, products normally supplied, internal remarks.
- **Actor:** Super Admin.
- **Preconditions:** Admin authenticated; KYC documents available where required.
- **Inputs:** Farmer master data.
- **Processing/business logic:** Persist farmer record; assign Farmer ID (FR-FRM-002); link mobile (FR-FRM-003).
- **Outputs:** Farmer Master record.
- **Validation:** Required fields per `[SOURCE §3]`; mobile format valid.
- **Error handling:** Missing/invalid data → block save with field-level errors.
- **Permissions:** Admin full access (`[SOURCE §2]`).
- **Audit requirement:** Creation logged (`[SOURCE §17]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §3]`.

### FR-FRM-002 — Permanent unique Farmer ID
- **Requirement:** Each farmer MUST receive a permanent unique Farmer ID (e.g., F-0001).
- **Description:** The ID is the permanent primary business identity.
- **Actor:** System.
- **Preconditions:** Farmer record is being created.
- **Inputs:** Automatic ID generation.
- **Processing/business logic:** Generate unique ID; enforce uniqueness at data level.
- **Outputs:** Assigned Farmer ID.
- **Validation:** Uniqueness must hold.
- **Error handling:** Collision → regenerate (uniqueness guaranteed).
- **Permissions:** System-assigned.
- **Audit requirement:** Assignment logged.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §3]`.

### FR-FRM-003 — Mobile linked for WhatsApp identity
- **Requirement:** The registered mobile MUST be linked to the Farmer ID for WhatsApp identification.
- **Description:** Enables notifications and Phase 2 chatbot identity resolution.
- **Actor:** System.
- **Preconditions:** Farmer record with mobile.
- **Inputs:** Registered mobile number.
- **Processing/business logic:** Store/link mobile to Farmer ID.
- **Outputs:** Linked identity.
- **Validation:** Valid mobile format.
- **Error handling:** Duplicate-mobile scenario handling undefined (OQ-13).
- **Permissions:** Admin manages.
- **Audit requirement:** Linkage changes logged.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §3, §21]`.

### FR-FRM-004 — KYC capture
- **Requirement:** The system MUST capture KYC information/documents where required.
- **Description:** Farmer verification data per business KYC rules.
- **Actor:** Super Admin.
- **Preconditions:** KYC required for the farmer.
- **Inputs:** KYC documents/details.
- **Processing/business logic:** Store KYC data; allowed document types/validation undefined (OQ-10).
- **Outputs:** KYC record.
- **Validation:** Per business KYC policy (undefined).
- **Error handling:** Missing KYC → flagged per policy (undefined).
- **Permissions:** Admin; restricted access to sensitive data (`[SOURCE §18]`).
- **Audit requirement:** KYC changes logged (`[SOURCE §17]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §3]`.

### FR-FRM-005 — Bank details with restricted access
- **Requirement:** Bank details MUST be captured and access restricted.
- **Description:** Bank name, account number, IFSC stored; sensitive access restricted.
- **Actor:** Super Admin.
- **Preconditions:** Farmer record exists.
- **Inputs:** Bank details.
- **Processing/business logic:** Store; enforce restricted access (`[SOURCE §18]`); encryption for sensitive data.
- **Outputs:** Bank data on record.
- **Validation:** Format checks (IFSC/account).
- **Error handling:** Invalid bank data → validation error.
- **Permissions:** Restricted to authorised roles (`[SOURCE §18]`).
- **Audit requirement:** Access/modification logged.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §3, §18]`.

### FR-FRM-006 — Status, products and remarks
- **Requirement:** Farmer records MUST capture status, products normally supplied, and internal remarks.
- **Description:** Operational context fields for the farmer relationship.
- **Actor:** Super Admin.
- **Preconditions:** Farmer record exists.
- **Inputs:** Status, products supplied, remarks.
- **Processing/business logic:** Store values; status transitions undefined (OQ).
- **Outputs:** Updated farmer master.
- **Validation:** Status value valid (enum undefined; Proposed Enhancement).
- **Error handling:** Invalid status → error.
- **Permissions:** Admin.
- **Audit requirement:** Changes logged (`[SOURCE §17]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §3]`.

### FR-FRM-007 — Farmer lookup for procurement
- **Requirement:** Employees MUST be able to select/search farmers for procurement.
- **Description:** Guided entry starts with farmer identification.
- **Actor:** Procurement Employee.
- **Preconditions:** Employee authenticated; farmer exists.
- **Inputs:** Search criteria (ID/name/mobile).
- **Processing/business logic:** Match farmer; return farmer info required for procurement only (`[SOURCE §16]`).
- **Outputs:** Farmer selection options.
- **Validation:** Match against Farmer ID/record.
- **Error handling:** Not found → no match; guidance undefined.
- **Permissions:** Employee — limited info scope (`[SOURCE §16]`); Phase 2 area restriction (`[SOURCE §22]`).
- **Audit requirement:** Lookup events logged as applicable (`[SOURCE §17]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §5, §16]`.

---

## 2.3 Employee

### FR-EMP-001 — Employee master creation
- **Requirement:** The system MUST allow creation of Employee Master records.
- **Description:** Fields: Employee ID, name, mobile, email, authentication credentials, role, status.
- **Actor:** Super Admin.
- **Preconditions:** Admin authenticated.
- **Inputs:** Employee data.
- **Processing/business logic:** Persist employee record; assign Employee ID (FR-EMP-002).
- **Outputs:** Employee Master record.
- **Validation:** Required fields per `[SOURCE §4]`.
- **Error handling:** Missing/invalid data → block with errors.
- **Permissions:** Admin full access (`[SOURCE §2]`).
- **Audit requirement:** Creation logged (`[SOURCE §17]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §4]`.

### FR-EMP-002 — Employee ID assignment
- **Requirement:** Each employee MUST receive a unique Employee ID (e.g., EMP-001).
- **Description:** Stable identity for logins and transaction attribution.
- **Actor:** System; Super Admin.
- **Preconditions:** Employee record being created.
- **Inputs:** Employee name.
- **Processing/business logic:** Generate/assign unique Employee ID.
- **Outputs:** Employee ID.
- **Validation:** Uniqueness.
- **Error handling:** Collision prevented.
- **Permissions:** Admin.
- **Audit requirement:** Logged.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §4]`.

### FR-EMP-003 — Credentials, role and status
- **Requirement:** Employee records MUST carry authentication credentials, role and status.
- **Description:** Enables login and access scoping.
- **Actor:** Super Admin.
- **Preconditions:** Employee record exists.
- **Inputs:** Credentials, role, status.
- **Processing/business logic:** Store securely; role drives permissions (`[SOURCE §16]`); status gates login.
- **Outputs:** Enabled employee account.
- **Validation:** Role/status values valid (catalogue undefined — OQ-09).
- **Error handling:** Invalid role → error.
- **Permissions:** Admin.
- **Audit requirement:** Credential/role/status changes logged (`[SOURCE §17]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §4]`.

### FR-EMP-004 — Reserved area/location field
- **Requirement:** The Employee Master MUST reserve an area/location field for Phase 2.
- **Description:** Field exists but stays inactive until area management (Phase 2).
- **Actor:** System.
- **Preconditions:** Data model supported.
- **Inputs:** None in Phase 1.
- **Processing/business logic:** Capability reserved; not operational in Phase 1 (`[SOURCE §20]`).
- **Outputs:** Reserved schema field.
- **Validation:** N/A (inactive).
- **Error handling:** N/A.
- **Permissions:** Admin (Phase 2).
- **Audit requirement:** Logged when used (Phase 2).
- **Priority:** MUST — **Phase:** 1 (reserved) / 2 (active) — **Source:** `[SOURCE §4, §20, §23]`.

### FR-EMP-005 — Transaction attribution
- **Requirement:** Every transaction MUST record the creating employee.
- **Description:** Backend trail on purchases/invoices.
- **Actor:** System.
- **Preconditions:** Employee performs a transaction.
- **Inputs:** Transaction + employee identity.
- **Processing/business logic:** Persist employee ID with transaction.
- **Outputs:** Attributed transaction.
- **Validation:** Employee ID present.
- **Error handling:** Missing identity → block transaction.
- **Permissions:** By role.
- **Audit requirement:** Core of audit trail (`[SOURCE §17]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §2]`.

### FR-EMP-006 — Scale support
- **Requirement:** Employee scale MUST support 100 (initial) toward 500 without redesign.
- **Description:** Login/operation capacity target.
- **Actor:** System.
- **Preconditions:** Growing employee base.
- **Inputs:** N/A (architecture).
- **Processing/business logic:** Design for scale (`[SOURCE §19]`).
- **Outputs:** Continued operation at scale.
- **Validation:** Capacity tests (parameters undefined).
- **Error handling:** Capacity pressure handled without fundamental redesign.
- **Permissions:** N/A.
- **Audit requirement:** N/A.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §2, §19]`.

---

## 2.4 Procurement

### FR-PROC-001 — Guided purchase workflow
- **Requirement:** Procurement MUST follow: Login → Select/Search Farmer → Select Product → Enter Weight → Enter Rate → Calculate Amount → Confirm → Generate Invoice.
- **Description:** Standardised field purchase entry.
- **Actor:** Procurement Employee.
- **Preconditions:** Employee logged in.
- **Inputs:** Farmer, product, weight, rate, optional deduction, remarks.
- **Processing/business logic:** Sequence steps and enforce order (`[SOURCE §5]`).
- **Outputs:** Confirmed transaction; invoice trigger.
- **Validation:** Mandatory steps completed.
- **Error handling:** Missing step/farmer/product → block confirmation.
- **Permissions:** Employee (procurement); Admin full.
- **Audit requirement:** Creation logged with employee (`[SOURCE §2, §17]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §5]`.

### FR-PROC-002 — Transaction data fields
- **Requirement:** The transaction MUST record Farmer ID, farmer name, Employee ID, date/time, product, quantity/weight, unit, rate, gross amount, deduction (if applicable), net amount, remarks.
- **Description:** Full purchase record.
- **Actor:** System (persist); Employee (enter).
- **Preconditions:** Entry in progress.
- **Inputs:** See requirement fields.
- **Processing/business logic:** Capture and persist.
- **Outputs:** Stored transaction.
- **Validation:** Required fields present; amounts numeric.
- **Error handling:** Invalid input → field errors.
- **Permissions:** Employee/Admin.
- **Audit requirement:** Logged (`[SOURCE §17]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §5]`.

### FR-PROC-003 — Automatic amount calculation
- **Requirement:** Amount MUST be calculated automatically as Quantity × Rate.
- **Description:** Eliminates manual calculation errors.
- **Actor:** System.
- **Preconditions:** Quantity and rate entered.
- **Inputs:** Quantity, rate.
- **Processing/business logic:** gross = quantity × rate.
- **Outputs:** Gross amount.
- **Validation:** Non-negative numeric inputs.
- **Error handling:** Zero/invalid → error.
- **Permissions:** Automatic.
- **Audit requirement:** Automatic; logged with transaction.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §5]`.

### FR-PROC-004 — Deduction and net amount
- **Requirement:** A deduction, if applicable, MUST be recorded and net amount derived.
- **Description:** net = gross − deduction.
- **Actor:** Employee (enter); System (compute net).
- **Preconditions:** Deduction applies.
- **Inputs:** Deduction value.
- **Processing/business logic:** Compute net amount. Deduction types/approval rules undefined (OQ-06).
- **Outputs:** Net amount.
- **Validation:** Deduction ≤ gross (proposed guard).
- **Error handling:** Negative net → error.
- **Permissions:** Employee.
- **Audit requirement:** Logged.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §5]`.

### FR-PROC-005 — Product and unit selection
- **Requirement:** A product and unit MUST be selectable during entry.
- **Description:** Product reference feeds the purchase and invoice.
- **Actor:** Employee; System.
- **Preconditions:** Product catalogue available (catalogue definition undefined — OQ-05).
- **Inputs:** Product, unit.
- **Processing/business logic:** Present selectable product/unit.
- **Outputs:** Selected product/unit.
- **Validation:** Product must exist.
- **Error handling:** Unknown product → error.
- **Permissions:** Employee.
- **Audit requirement:** Logged.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §5]`; catalogue detail undefined (OQ-05).

### FR-PROC-006 — Remarks capture
- **Requirement:** Transactions MUST support remarks.
- **Description:** Free text context on the purchase.
- **Actor:** Employee.
- **Preconditions:** Entry in progress.
- **Inputs:** Remarks text.
- **Processing/business logic:** Store remarks with transaction.
- **Outputs:** Stored remark.
- **Validation:** Length limit (undefined; proposed).
- **Error handling:** Oversized text → error.
- **Permissions:** Employee.
- **Audit requirement:** Logged.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §5]`.

---

## 2.5 Invoice

### FR-INV-001 — Automatic invoice numbering
- **Requirement:** Invoice numbers MUST be generated automatically from Farmer ID + transaction sequence (e.g., F-0001-17).
- **Description:** Rule-based numbering.
- **Actor:** System.
- **Preconditions:** Confirmed transaction.
- **Inputs:** Farmer ID, next sequence value.
- **Processing/business logic:** Compose number = Farmer ID + '-' + sequence.
- **Outputs:** Unique invoice number.
- **Validation:** Uniqueness enforced.
- **Error handling:** Collision prevented at data level.
- **Permissions:** System-generated.
- **Audit requirement:** Numbering assigned; logged.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §6]`.

### FR-INV-002 — Separate DB fields
- **Requirement:** Farmer ID and transaction sequence MUST be stored as separate fields.
- **Description:** Enables per-farmer sequence control.
- **Actor:** System.
- **Preconditions:** Data model.
- **Inputs:** Farmer ID, sequence.
- **Processing/business logic:** Persist separately.
- **Outputs:** Stored components.
- **Validation:** Sequence per farmer independent (`[SOURCE §6]`).
- **Error handling:** Duplicate for farmer → block.
- **Permissions:** System.
- **Audit requirement:** Logged.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §6]`.

### FR-INV-003 — No manual numbering
- **Requirement:** Employees MUST NOT type invoice numbers.
- **Description:** Preserves numbering integrity.
- **Actor:** System.
- **Preconditions:** Invoice generation path.
- **Inputs:** Automatically derived components only.
- **Processing/business logic:** No manual entry surface.
- **Outputs:** System-assigned number.
- **Validation:** Employee input blocked.
- **Error handling:** N/A by design.
- **Permissions:** System.
- **Audit requirement:** Enforced.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §6]`.

### FR-INV-004 — Cancellation retention (no reuse)
- **Requirement:** Cancelled invoices MUST remain in the system and their numbers MUST NOT be reused.
- **Description:** History and numbering integrity preserved.
- **Actor:** Authorised user (actor undefined — OQ-08); System.
- **Preconditions:** Invoice exists.
- **Inputs:** Cancellation action.
- **Processing/business logic:** Mark cancelled; retain record; never reissue number (`[SOURCE §6]`).
- **Outputs:** Cancelled invoice; audited change.
- **Validation:** State transition valid.
- **Error handling:** Re-cancellation handled by state.
- **Permissions:** Actor undefined (OQ-08).
- **Audit requirement:** Mandatory audit (`[SOURCE §6, §17]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §6]`.

### FR-INV-005 — Transaction confirmation display
- **Requirement:** On generation, the system MUST display Farmer, Invoice, Date, Product, Quantity, Rate, Total, Employee.
- **Description:** Post-save confirmation.
- **Actor:** System; Employee.
- **Preconditions:** Invoice generated.
- **Inputs:** Invoice record.
- **Processing/business logic:** Render confirmation.
- **Outputs:** Confirmation view.
- **Validation:** Data present.
- **Error handling:** Display graceful if data missing.
- **Permissions:** Creator/admin.
- **Audit requirement:** Logged.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §7]`.

### FR-INV-006 — Invoice modification audit
- **Requirement:** All invoice modifications MUST be recorded in the audit log.
- **Description:** Every change traceable.
- **Actor:** System.
- **Preconditions:** Invoice changed.
- **Inputs:** Change details (original/new).
- **Processing/business logic:** Write audit entry with original and new values.
- **Outputs:** Audit record.
- **Validation:** Both values captured.
- **Error handling:** Logging failure must not lose trail (design requirement).
- **Permissions:** System-level.
- **Audit requirement:** This requirement is itself an audit rule (`[SOURCE §6, §17]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §6, §17]`.

---

## 2.6 Payment

### FR-PAY-001 — Payment record
- **Requirement:** Payments MUST capture Farmer ID, invoice number/allocation, payment amount, payment date, payment status, payment mode, bank reference, UTR, remarks.
- **Description:** Complete payment capture.
- **Actor:** Payment creator (role undefined; accounts staff implied by `[SOURCE §11]`).
- **Preconditions:** Invoice exists.
- **Inputs:** Payment fields.
- **Processing/business logic:** Persist; allocate to invoice(s).
- **Outputs:** Payment record.
- **Validation:** Amount > 0; allocation ≤ due (proposed guard).
- **Error handling:** Over-allocation → block (proposed).
- **Permissions:** Accounts/Admin (undefined formally).
- **Audit requirement:** Logged (`[SOURCE §17]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §10]`.

### FR-PAY-002 — Full payment
- **Requirement:** Full payment against an invoice MUST be supported.
- **Description:** Settles an invoice completely.
- **Actor:** Payment creator.
- **Preconditions:** Invoice due.
- **Inputs:** Full amount.
- **Processing/business logic:** Mark invoice fully paid.
- **Outputs:** Paid invoice.
- **Validation:** Amount = remaining due.
- **Error handling:** Mismatch → error.
- **Permissions:** As FR-PAY-001.
- **Audit requirement:** Logged.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §10]`.

### FR-PAY-003 — Partial payment
- **Requirement:** Partial payment against an invoice MUST be supported.
- **Description:** Invoice remains partially outstanding.
- **Actor:** Payment creator.
- **Preconditions:** Invoice due.
- **Inputs:** Partial amount.
- **Processing/business logic:** Reduce outstanding; track partial status.
- **Outputs:** Partially paid invoice.
- **Validation:** 0 < amount < due.
- **Error handling:** Amount ≥ due → error.
- **Permissions:** As FR-PAY-001.
- **Audit requirement:** Logged.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §10]`.

### FR-PAY-004 — Multiple payments per invoice
- **Requirement:** Multiple payments against one invoice MUST be supported.
- **Description:** Instalment settlement.
- **Actor:** Payment creator.
- **Preconditions:** Invoice outstanding.
- **Inputs:** Successive payments.
- **Processing/business logic:** Accumulate; close when sum = total.
- **Outputs:** Settled invoice (on completion).
- **Validation:** Cumulative ≤ total.
- **Error handling:** Over-payment → block/flag (proposed).
- **Permissions:** As FR-PAY-001.
- **Audit requirement:** Logged.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §10]`.

### FR-PAY-005 — Multiple invoices per payment
- **Requirement:** One payment against multiple invoices MUST be supported.
- **Description:** Single payout clears several dues.
- **Actor:** Payment creator.
- **Preconditions:** Multiple invoices due (same farmer).
- **Inputs:** Payment + allocations.
- **Processing/business logic:** Distribute amounts across invoices.
- **Outputs:** Updated invoice allocations.
- **Validation:** Sum of allocations = payment.
- **Error handling:** Sum mismatch → error.
- **Permissions:** As FR-PAY-001.
- **Audit requirement:** Logged.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §10]`.

---

## 2.7 Reconciliation

### FR-REC-001 — Banking/API integration flow
- **Requirement:** The system MUST follow: Portal → Bank/API → Payment → UTR/status → Portal.
- **Description:** Automated payment status pipeline.
- **Actor:** System; Banking/API provider.
- **Preconditions:** Integration configured (provider undefined — OQ-03).
- **Inputs:** Payment initiation; provider responses.
- **Processing/business logic:** Route and receive payment status.
- **Outputs:** Payment status/UTR in portal.
- **Validation:** Provider responses validated.
- **Error handling:** Provider unavailable → pending/failed state.
- **Permissions:** System (secure API auth `[SOURCE §18]`).
- **Audit requirement:** Integration activity logged (`[SOURCE §17]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §11]`.

### FR-REC-002 — Automatic UTR/status receipt
- **Requirement:** Wherever supported, the system MUST automatically receive payment status and UTR.
- **Description:** Minimises manual reconciliation.
- **Actor:** System.
- **Preconditions:** Provider supports return (fully/partially).
- **Inputs:** Provider callback/poll data.
- **Processing/business logic:** Capture status + UTR; attach to payment (FR-WH map to payment notification).
- **Outputs:** Updated payment record.
- **Validation:** Consistent payment reference.
- **Error handling:** Unmatched/invalid → queue (FR-REC-004).
- **Permissions:** System.
- **Audit requirement:** Logged.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §11]`.

### FR-REC-003 — Payment statuses
- **Requirement:** The system MUST maintain statuses: matched, unmatched, failed, pending, duplicate.
- **Description:** Reconcilable payment states.
- **Actor:** System.
- **Preconditions:** Payment processing.
- **Inputs:** Payment/provider signals.
- **Processing/business logic:** Classify status per `[SOURCE §11]`.
- **Outputs:** Statused payments.
- **Validation:** Status transitions valid.
- **Error handling:** Ambiguous → unmatched/queue.
- **Permissions:** Accounts/admin.
- **Audit requirement:** Status changes logged.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §11]`.

### FR-REC-004 — Exception/Reconciliation Queue
- **Requirement:** The system MUST provide an Exception/Reconciliation Queue for accounts staff.
- **Description:** Worklist for unmatched/failed/pending/duplicate items.
- **Actor:** Accounts staff.
- **Preconditions:** Items exist in abnormal states.
- **Inputs:** Queue items.
- **Processing/business logic:** Present; allow resolution actions.
- **Outputs:** Resolved items; audit.
- **Validation:** Actions valid for state.
- **Error handling:** Resolution rules undefined (OQ).

- **Permissions:** Accounts staff (`[SOURCE §11]`).
- **Audit requirement:** Queue actions logged (`[SOURCE §17]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §11]`.

### FR-REC-005 — Automatic ledger update on match
- **Requirement:** On confirmed/matched payment, the ledger MUST update automatically.
- **Description:** Keeps ledger current.
- **Actor:** System.
- **Preconditions:** Payment matched.
- **Inputs:** Confirmed payment.
- **Processing/business logic:** Post payment to farmer ledger.
- **Outputs:** Updated ledger.
- **Validation:** Posting consistent.
- **Error handling:** Posting failure surfaced.
- **Permissions:** Automatic.
- **Audit requirement:** Logged; no silent overwrite.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §11]`.

---

## 2.8 Ledger

### FR-LED-001 — Farmer-wise ledger
- **Requirement:** The system MUST maintain a complete farmer-wise ledger.
- **Description:** Per-farmer financial record.
- **Actor:** System.
- **Preconditions:** Farmer transactions exist.
- **Inputs:** Invoice postings, payments.
- **Processing/business logic:** Store/ maintain per farmer (`[SOURCE §9]`).
- **Outputs:** Farmer ledger.
- **Validation:** Entries traceable to sources.
- **Error handling:** Data integrity checks.
- **Permissions:** Admin full; employee scoped; farmer own (portal).
- **Audit requirement:** Traceable; no silent overwrite (`[SOURCE §17]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §9]`.

### FR-LED-002 — Ledger content
- **Requirement:** The ledger MUST show date, invoice, product, quantity, rate, amount, payment, UTR.
- **Description:** Required ledger columns.
- **Actor:** System.
- **Preconditions:** Entries exist.
- **Inputs:** Transaction/payment data.
- **Processing/business logic:** Present columns per `[SOURCE §9]`.
- **Outputs:** Ledger view.
- **Validation:** Data present.
- **Error handling:** Rendering fallbacks.
- **Permissions:** Per role.
- **Audit requirement:** Read access logged as required.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §9]`.

### FR-LED-003 — No silent overwrite
- **Requirement:** Historical financial records MUST NOT be silently overwritten.
- **Description:** Integrity rule on all corrections.
- **Actor:** System.
- **Preconditions:** Correction attempted.
- **Inputs:** Change request.
- **Processing/business logic:** Apply corrections via audit-tracked change, preserving history (`[SOURCE §17]`).
- **Outputs:** Audited correction.
- **Validation:** Original value preserved.
- **Error handling:** Silent overwrite prevented.
- **Permissions:** Authorised roles only.
- **Audit requirement:** Mandatory (`[SOURCE §17]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §17]`.

### FR-LED-004 — Outstanding balance
- **Requirement:** The ledger SHALL make outstanding balance derivable per farmer.
- **Description:** Basis for reports/dashboard/outstanding queries.
- **Actor:** System.
- **Preconditions:** Ledger data.
- **Inputs:** Invoices, payments.
- **Processing/business logic:** outstanding = invoiced − paid.
- **Outputs:** Outstanding figure.
- **Validation:** Computed consistently.
- **Error handling:** N/A.
- **Permissions:** Per role.
- **Audit requirement:** N/A (computed).
- **Priority:** MUST — **Phase:** 1 — **Source:** Derived from `[SOURCE §9, §14, §15]`.

---

## 2.9 Statements

### FR-MST-001 — Scheduled generation
- **Requirement:** On the 1st day of each month, the system MUST automatically generate the previous month's statement.
- **Description:** e.g., 1 Sep generates 1–31 Aug.
- **Actor:** System (scheduled).
- **Preconditions:** Prior month concluded.
- **Inputs:** Ledger/payment data for period.
- **Processing/business logic:** Aggregate period data per `[SOURCE §13]`.
- **Outputs:** Statement record.
- **Validation:** Period boundaries.
- **Error handling:** Generation failure → failed/retry status (`[SOURCE §13]`).
- **Permissions:** System job.
- **Audit requirement:** Generation logged; status recorded.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §13]`.

### FR-MST-002 — Statement content
- **Requirement:** Statements MUST include Farmer ID and name, period, opening balance (if applicable), all purchase invoices (product, quantity, rate, amount), all payments (dates + UTRs), closing/outstanding balance.
- **Description:** Mandated content.
- **Actor:** System.
- **Preconditions:** Data available.
- **Inputs:** Period data.
- **Processing/business logic:** Compose content per `[SOURCE §13]`.
- **Outputs:** Statement data set.
- **Validation:** All sections present.
- **Error handling:** Incomplete data → status handling.
- **Permissions:** System.
- **Audit requirement:** Logged.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §13]`.

### FR-MST-003 — PDF generation
- **Requirement:** Statements MUST be generated as PDF.
- **Description:** Deliverable format.
- **Actor:** System.
- **Preconditions:** Statement data.
- **Inputs:** Statement data set.
- **Processing/business logic:** Render PDF.
- **Outputs:** PDF artifact.
- **Validation:** Valid PDF.
- **Error handling:** Render error → failed/retry.
- **Permissions:** System.
- **Audit requirement:** Status recorded.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §13]`.

### FR-MST-004 — WhatsApp delivery
- **Requirement:** Statements MUST be sent automatically via WhatsApp.
- **Description:** Delivery channel.
- **Actor:** System.
- **Preconditions:** Farmer mobile valid; provider configured.
- **Inputs:** PDF + farmer mobile.
- **Processing/business logic:** Send message; capture response.
- **Outputs:** Delivery attempt + status.
- **Validation:** Number is WhatsApp-valid.
- **Error handling:** Delivery failure → failed/retry (`[SOURCE §13]`).
- **Permissions:** System.
- **Audit requirement:** Delivery statuses recorded.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §13]`.

### FR-MST-005 — Status recording
- **Requirement:** Statement statuses MUST be recorded: generated, sent, delivered, failed, retry.
- **Description:** Operational traceability.
- **Actor:** System.
- **Preconditions:** Statement lifecycle.
- **Inputs:** Lifecycle events.
- **Processing/business logic:** Track transitions.
- **Outputs:** Status history.
- **Validation:** Valid transitions.
- **Error handling:** Retry policy undefined (OQ-14).
- **Permissions:** System/admin.
- **Audit requirement:** Logged.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §13]`.

---

## 2.10 WhatsApp

### FR-WH-001 — Purchase notification
- **Requirement:** After invoice creation, the system MUST automatically send a WhatsApp purchase notification.
- **Description:** Summarises purchase/invoice details.
- **Actor:** System.
- **Preconditions:** Invoice created (`[SOURCE §5]`).
- **Inputs:** Invoice data, farmer mobile.
- **Processing/business logic:** Compose and send notification.
- **Outputs:** Notification sent.
- **Validation:** Mobile linked (`[SOURCE §3]`).
- **Error handling:** Send failure — retry policy undefined for this type (OQ-04).
- **Permissions:** System.
- **Audit requirement:** Delivery logged (Proposed: full status tracking).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §8]`.

### FR-WH-002 — Payment notification
- **Requirement:** When payment is confirmed, the system MUST automatically send a WhatsApp payment notification.
- **Description:** Includes amount and UTR.
- **Actor:** System.
- **Preconditions:** Bank/API confirms payment (`[SOURCE §11]`).
- **Inputs:** Payment + UTR, farmer mobile.
- **Processing/business logic:** Compose and send.
- **Outputs:** Notification sent.
- **Validation:** Confirmed payment.
- **Error handling:** Send failure handling undefined (OQ-04).
- **Permissions:** System.
- **Audit requirement:** Logged.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §12]`.

### FR-WH-003 — Statement notification
- **Requirement:** Monthly statements MUST be delivered via WhatsApp.
- **Description:** Statement delivery mechanism.
- **Actor:** System.
- **Preconditions:** Statement generated (FR-MST-003).
- **Inputs:** PDF, mobile.
- **Processing/business logic:** Send; record status.
- **Outputs:** Delivery + status.
- **Validation:** Per FR-MST-004.
- **Error handling:** Failed → retry state.
- **Permissions:** System.
- **Audit requirement:** Statuses recorded (FR-MST-005).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §13]`.

### FR-WH-004 — Farmer identification by mobile
- **Requirement:** Farmers MUST be identified on WhatsApp by the registered mobile linked to the Farmer ID.
- **Description:** Identity resolution for outbound and Phase 2 chatbot.
- **Actor:** System.
- **Preconditions:** Mobile linked (`[SOURCE §3]`).
- **Inputs:** Mobile number.
- **Processing/business logic:** Resolve Farmer ID.
- **Outputs:** Identified farmer.
- **Validation:** Linkage valid.
- **Error handling:** Unlinked number → no identity (handling undefined).
- **Permissions:** System.
- **Audit requirement:** Logged.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §3, §21]`.

---

## 2.11 Farmer Portal `[NEW]`

All requirements in this section are **newly approved (`[NEW]`)** ; not part of the original PDF. Behavioural detail not defined is marked.

### FR-PRT-001 — Farmer Login
- **Requirement:** The portal MUST provide Farmer Login (`[NEW]`).
- **Description:** Farmer authentication for self-service.
- **Actor:** Farmer; System.
- **Preconditions:** Portal account exists.
- **Inputs:** Credentials/OTP (method undefined; OTP = Proposed Enhancement).
- **Processing/business logic:** Authenticate; scope session to farmer.
- **Outputs:** Authenticated session.
- **Validation:** Per adopted method (OQ-02).
- **Error handling:** Denial + retry controls.
- **Permissions:** Farmer — own data.
- **Audit requirement:** Login logging (Proposed).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[NEW]`.

### FR-PRT-002 — Responsive web, no native app
- **Requirement:** The portal MUST be responsive web/mobile-browser based with no native mobile app in Phase 1 (`[NEW]`; `[SOURCE §1]`).
- **Description:** Accessible on browsers.
- **Actor:** System.
- **Preconditions:** Browser access.
- **Inputs:** Page request.
- **Processing/business logic:** Render responsively.
- **Outputs:** Usable UI.
- **Validation:** Desktop/mobile rendering.
- **Error handling:** Graceful degradation.
- **Permissions:** Logged-in farmer.
- **Audit requirement:** N/A.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[NEW]` (N-03).

### FR-PRT-003 — Farmer Dashboard
- **Requirement:** The portal MUST provide a Dashboard of the farmer's own position.
- **Description:** Summary of purchases, payments, outstanding (widgets undefined OQ-01).
- **Actor:** Farmer.
- **Preconditions:** Authenticated.
- **Inputs:** Own data.
- **Processing/business logic:** Aggregate own data.
- **Outputs:** Dashboard view.
- **Validation:** Own-data only.
- **Error handling:** Empty-state handling undefined.
- **Permissions:** Self.
- **Audit requirement:** Not specified (view).
- **Priority:** SHOULD — **Phase:** 1 — **Source:** `[NEW]`.

### FR-PRT-004 — My Profile
- **Requirement:** The portal MUST show the farmer's own profile.
- **Description:** Master data incl. bank/KYC (edit scope undefined OQ-18).
- **Actor:** Farmer.
- **Preconditions:** Authenticated.
- **Inputs:** Own master data.
- **Processing/business logic:** Present read-only (edits undefined).
- **Outputs:** Profile view.
- **Validation:** Own data only.
- **Error handling:** Sensitive-data masking rules undefined.
- **Permissions:** Self.
- **Audit requirement:** Not specified.
- **Priority:** SHOULD — **Phase:** 1 — **Source:** `[NEW]`.

### FR-PRT-005 — My Purchases
- **Requirement:** The portal MUST list the farmer's own purchases.
- **Description:** Purchase history.
- **Actor:** Farmer.
- **Preconditions:** Authenticated; data exists.
- **Inputs:** Own purchase data.
- **Processing/business logic:** List own purchases.
- **Outputs:** Purchase list.
- **Validation:** Own data only.
- **Error handling:** Empty state undefined.
- **Permissions:** Self.
- **Audit requirement:** Not specified.
- **Priority:** SHOULD — **Phase:** 1 — **Source:** `[NEW]`.

### FR-PRT-006 — My Invoices
- **Requirement:** The portal MUST list the farmer's own invoices.
- **Description:** Own invoices + status.
- **Actor:** Farmer.
- **Preconditions:** Authenticated; invoices exist.
- **Inputs:** Own invoice data.
- **Processing/business logic:** List own invoices.
- **Outputs:** Invoice list/view.
- **Validation:** Own data only.
- **Error handling:** Empty state undefined.
- **Permissions:** Self.
- **Audit requirement:** Not specified.
- **Priority:** SHOULD — **Phase:** 1 — **Source:** `[NEW]`.

### FR-PRT-007 — My Payments
- **Requirement:** The portal MUST list the farmer's own payments.
- **Description:** Date, mode, UTR, status.
- **Actor:** Farmer.
- **Preconditions:** Authenticated; payments exist.
- **Inputs:** Own payment data.
- **Processing/business logic:** List own payments.
- **Outputs:** Payment list.
- **Validation:** Own data only.
- **Error handling:** Empty state undefined.
- **Permissions:** Self.
- **Audit requirement:** Not specified.
- **Priority:** SHOULD — **Phase:** 1 — **Source:** `[NEW]`.

### FR-PRT-008 — My Ledger
- **Requirement:** The portal MUST show the farmer's own ledger.
- **Description:** Per `[SOURCE §9]` content + outstanding.
- **Actor:** Farmer.
- **Preconditions:** Authenticated; ledger exists.
- **Inputs:** Own ledger data.
- **Processing/business logic:** Present own ledger.
- **Outputs:** Ledger view.
- **Validation:** Own data only.
- **Error handling:** Empty state undefined.
- **Permissions:** Self.
- **Audit requirement:** Not specified.
- **Priority:** SHOULD — **Phase:** 1 — **Source:** `[NEW]` (content `[SOURCE §9]`).

### FR-PRT-009 — My Statements
- **Requirement:** The portal MUST allow the farmer to view/download own statements.
- **Description:** Statement PDFs generated per FR-MST.
- **Actor:** Farmer.
- **Preconditions:** Authenticated; statements generated.
- **Inputs:** Own statement artifacts.
- **Processing/business logic:** List/provide PDFs.
- **Outputs:** Statement access.
- **Validation:** Own data only.
- **Error handling:** Not generated → empty state undefined.
- **Permissions:** Self.
- **Audit requirement:** Not specified.
- **Priority:** SHOULD — **Phase:** 1 — **Source:** `[NEW]` (content `[SOURCE §13]`).

### FR-PRT-010 — Notifications
- **Requirement:** The portal MUST show the farmer's own notification history.
- **Description:** Purchase, payment, statement notifications.
- **Actor:** Farmer.
- **Preconditions:** Authenticated; notifications exist.
- **Inputs:** Own notification records.
- **Processing/business logic:** Present history.
- **Outputs:** Notification view.
- **Validation:** Own data only.
- **Error handling:** Empty state undefined.
- **Permissions:** Self.
- **Audit requirement:** Not specified.
- **Priority:** SHOULD — **Phase:** 1 — **Source:** `[NEW]`.

### FR-PRT-011 — Support
- **Requirement:** The portal MUST provide a Support entry point.
- **Description:** Contact channel (channel undefined OQ-16).
- **Actor:** Farmer.
- **Preconditions:** Authenticated.
- **Inputs:** Support navigation.
- **Processing/business logic:** Present channel.
- **Outputs:** Support contact.
- **Validation:** N/A.
- **Error handling:** Channel unavailable — undefined.
- **Permissions:** Farmer.
- **Audit requirement:** Contact events (Proposed).
- **Priority:** SHOULD — **Phase:** 1 — **Source:** `[NEW]`.

### FR-PRT-012 — Own-data enforcement
- **Requirement:** A farmer MUST only ever access their own data (backend enforced).
- **Description:** Data isolation security rule.
- **Actor:** System.
- **Preconditions:** Farmer session.
- **Inputs:** Any data request.
- **Processing/business logic:** Enforce farmer-scoped queries at backend.
- **Outputs:** Own-data results only.
- **Validation:** Scope check on every query.
- **Error handling:** Cross-scope attempt blocked + logged (Proposed).
- **Permissions:** Farmer — self.
- **Audit requirement:** Logging of denied cross-scope attempts (Proposed).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[NEW]` (derived from `[SOURCE §22]` backend-enforcement philosophy).

---

## 2.12 Reports

### FR-RPT-001 — Farmer reports
- **Requirement:** Farmer-wise purchase, payment, outstanding, ledger and monthly statement reports MUST be available.
- **Description:** Per-farmer analytical views.
- **Actor:** Super Admin.
- **Preconditions:** Data exists; user authorised.
- **Inputs:** Farmer selection, filters.
- **Processing/business logic:** Aggregate per `[SOURCE §14]`.
- **Outputs:** Report.
- **Validation:** Filters valid.
- **Error handling:** No data → empty report.
- **Permissions:** Admin full (`[SOURCE §2]`); configurable else (`[SOURCE §16]`).
- **Audit requirement:** Report access logging (Proposed).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §14]`.

### FR-RPT-002 — Procurement reports
- **Requirement:** Date-wise, product-wise, employee-wise, farmer-wise, quantity-wise and rate-wise procurement reports MUST be available.
- **Description:** Multi-dimensional buying analysis.
- **Actor:** Super Admin.
- **Preconditions:** Procurement data.
- **Inputs:** Dimension + filters.
- **Processing/business logic:** Slice procurement data.
- **Outputs:** Report.
- **Validation:** Valid dimensions.
- **Error handling:** No data → empty.
- **Permissions:** Admin.
- **Audit requirement:** Proposed.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §14]`.

### FR-RPT-003 — Payment reports
- **Requirement:** Payment register, UTR register, paid, partially paid, pending and unreconciled reports MUST be available.
- **Description:** Payment visibility and reconciliation support.
- **Actor:** Super Admin.
- **Preconditions:** Payment data.
- **Inputs:** Report type + filters.
- **Processing/business logic:** Aggregate per `[SOURCE §14]`.
- **Outputs:** Report.
- **Validation:** Valid type.
- **Error handling:** No data → empty.
- **Permissions:** Admin.
- **Audit requirement:** Proposed.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §14]`.

### FR-RPT-004 — Export
- **Requirement:** Reports MUST be exportable to Excel, PDF and CSV where appropriate.
- **Description:** Standard export formats.
- **Actor:** Super Admin.
- **Preconditions:** Report generated.
- **Inputs:** Format selection.
- **Processing/business logic:** Export.
- **Outputs:** Export file.
- **Validation:** Format supported.
- **Error handling:** Export failure surfaced.
- **Permissions:** Admin.
- **Audit requirement:** Export logging (Proposed).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §14]`.

---

## 2.13 Dashboard

### FR-DSH-001 — KPI dashboard
- **Requirement:** The Admin Dashboard MUST show total farmers, active farmers, total employees, today's procurement, today's procurement value, monthly procurement, pending farmer payments, payments made, total outstanding, number of invoices, unreconciled bank transactions.
- **Description:** Mandated KPI set.
- **Actor:** Super Admin.
- **Preconditions:** Operational data.
- **Inputs:** KPI aggregation.
- **Processing/business logic:** Compute KPIs per `[SOURCE §15]`.
- **Outputs:** Dashboard.
- **Validation:** Figures consistent with source data.
- **Error handling:** Data gaps handled gracefully.
- **Permissions:** Admin full (`[SOURCE §2]`).
- **Audit requirement:** View events (Proposed).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §15]`.

### FR-DSH-002 — Date filters
- **Requirement:** The Admin Dashboard MUST support date filters.
- **Description:** KPI scoping by date.
- **Actor:** Super Admin.
- **Preconditions:** Dashboard open.
- **Inputs:** Date range.
- **Processing/business logic:** Filter KPIs.
- **Outputs:** Filtered dashboard.
- **Validation:** Valid range.
- **Error handling:** Invalid range → error.
- **Permissions:** Admin.
- **Audit requirement:** Proposed.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §15]`.

---

## 2.14 Audit

### FR-AUD-001 — Audit trail capture
- **Requirement:** The system MUST record user/employee ID, date and time, action, original value, new value and record affected.
- **Description:** Standard audit fields.
- **Actor:** System.
- **Preconditions:** Business action occurs.
- **Inputs:** Action context.
- **Processing/business logic:** Write audit entry.
- **Outputs:** Audit record.
- **Validation:** Fields consistent.
- **Error handling:** Audit failures preserved/flagged.
- **Permissions:** System write; admin read.
- **Audit requirement:** This is the audit mechanism (`[SOURCE §17]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §17]`.

### FR-AUD-002 — IP/device capture
- **Requirement:** IP/device information MUST be recorded where appropriate.
- **Description:** Context for forensic review.
- **Actor:** System.
- **Preconditions:** Action with device context.
- **Inputs:** IP/device metadata.
- **Processing/business logic:** Capture.
- **Outputs:** Enriched audit record.
- **Validation:** Where appropriate.
- **Error handling:** Missing metadata tolerated.
- **Permissions:** System.
- **Audit requirement:** Part of trail.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §17]`.

### FR-AUD-003 — No silent overwrite
- **Requirement:** Historical financial records MUST NOT be silently overwritten.
- **Description:** Corrections must be explicit and audited.
- **Actor:** System.
- **Preconditions:** Correction scenario.
- **Inputs:** Change request.
- **Processing/business logic:** Preserve original; record new value; log both.
- **Outputs:** Audited correction.
- **Validation:** Original preserved.
- **Error handling:** Silent overwrite prevented.
- **Permissions:** Authorised roles.
- **Audit requirement:** Mandatory.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §17]`.

### FR-AUD-004 — Admin audit review
- **Requirement:** Super Admin MUST be able to view audit logs.
- **Description:** Investigation capability.
- **Actor:** Super Admin.
- **Preconditions:** Admin authenticated.
- **Inputs:** Log query.
- **Processing/business logic:** Present audit data.
- **Outputs:** Audit log view.
- **Validation:** Access role.
- **Error handling:** Large volume handling.
- **Permissions:** Admin only (`[SOURCE §2]`).
- **Audit requirement:** Log view commands (Proposed).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §2, §17]`.

---

## 2.15 Settings

### FR-SET-001 — Settings management
- **Requirement:** Super Admin MUST be able to manage settings.
- **Description:** Central configuration.
- **Actor:** Super Admin.
- **Preconditions:** Admin authenticated.
- **Inputs:** Settings values.
- **Processing/business logic:** Apply configuration.
- **Outputs:** Active settings.
- **Validation:** Valid values.
- **Error handling:** Invalid → error.
- **Permissions:** Admin only (`[SOURCE §2]`).
- **Audit requirement:** Changes logged (`[SOURCE §17]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §2]`.

### FR-SET-002 — WhatsApp integration configuration
- **Requirement:** Super Admin MUST be able to manage WhatsApp integration.
- **Description:** Provider/credentials configuration (provider undefined OQ-04).
- **Actor:** Super Admin.
- **Preconditions:** Admin authenticated.
- **Inputs:** Provider settings.
- **Processing/business logic:** Configure channel.
- **Outputs:** Enabled WhatsApp channel.
- **Validation:** Credentials valid.
- **Error handling:** Invalid config surfaced.
- **Permissions:** Admin.
- **Audit requirement:** Logged.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §2]`.

### FR-SET-003 — Banking integration configuration
- **Requirement:** Super Admin MUST be able to manage banking integrations.
- **Description:** Provider/credentials for payment status flow (provider undefined OQ-03).
- **Actor:** Super Admin.
- **Preconditions:** Admin authenticated.
- **Inputs:** Bank settings.
- **Processing/business logic:** Configure integration.
- **Outputs:** Enabled banking integration.
- **Validation:** API credentials secure/auth (`[SOURCE §18]`).
- **Error handling:** Invalid config surfaced.
- **Permissions:** Admin.
- **Audit requirement:** Logged + secure storage.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §2, §18]`.

### FR-SET-004 — Configurable permission matrix
- **Requirement:** The permission matrix MUST be configurable by Admin.
- **Description:** Role/feature access control.
- **Actor:** Super Admin.
- **Preconditions:** Admin authenticated.
- **Inputs:** Matrix changes.
- **Processing/business logic:** Apply changes to RBAC (`FR-AUTH-004`).
- **Outputs:** Updated permissions.
- **Validation:** Valid roles/features.
- **Error handling:** Conflicts surfaced.
- **Permissions:** Admin.
- **Audit requirement:** Matrix changes logged (`[SOURCE §17]`).
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §16]`.

### FR-SET-005 — Backup and restoration
- **Requirement:** The system MUST perform regular automated backups and tested restoration.
- **Description:** Data protection (details under NFR/security).
- **Actor:** System.
- **Preconditions:** Schedule configured.
- **Inputs:** Database state.
- **Processing/business logic:** Automated backup; restoration testing.
- **Outputs:** Backups; test results.
- **Validation:** Restorable.
- **Error handling:** Failed backup surfaced.
- **Permissions:** System/admin.
- **Audit requirement:** Logged.
- **Priority:** MUST — **Phase:** 1 — **Source:** `[SOURCE §18]`.

---

## 3. Phase 2 Functional Outline (reference)

Not part of the Phase 1 FRS module list; captured for traceability.

| Ref | Capability | Source |
|---|---|---|
| FR-P2-01 | WhatsApp Chatbot: My Outstanding, My Ledger, My Purchases, My Payments, Last Payment, Last Purchase, Invoice Details, Statement, Payment Status | `[SOURCE §21]` |
| FR-P2-02 | Area management (create/manage areas) | `[SOURCE §23]` |
| FR-P2-03 | Area-based farmer allocation, reassignment; backend-enforced employee restriction | `[SOURCE §22, §23]` |
| FR-P2-04 | Area-based employee allocation, transfer | `[SOURCE §23]` |
| FR-P2-05 | Area-wise reporting (farmer count, procurement, payment, outstanding; employee performance; corrections & cancellations) | `[SOURCE §24]` |

---

## 4. Traceability Notes

- Every module requirement above is a Phase 1 functional requirement with the exception of the Phase 2 outline (§3).
- Undefined behaviours are referenced to open questions (PRD §28 / BRD §25): OQ-03 (bank provider), OQ-04 (WhatsApp provider/policy), OQ-05 (product catalogue), OQ-06 (deductions), OQ-07 (invoice output), OQ-08 (cancellation actor), OQ-09 (role catalogue), OQ-10 (KYC), OQ-13 (deduplication), OQ-14 (statement timing/retry), OQ-15/16/17/18 (portal: onboarding, support, language, edit scope).
- No requirement in this FRS contradicts the source PDF; all additions are confined to `[NEW]` Farmer Portal items and explicitly marked Proposed Enhancements.

---

*End of Functional Requirements Specification v1.0. Next in sequence: `07_API_SPECIFICATION.md` (or per index reading order).*