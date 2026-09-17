# Agri Procurement & Farmer Management System
## Farmer Portal Specification

---

## 1. Document Information

| Item | Value |
|---|---|
| Product name | Agri Procurement & Farmer Management System |
| Document | Farmer Portal Specification |
| Version | v1.0 |
| Status | Draft — aligned to PRD v1.0, BRD v1.0, Workflows v1.0, Modules v1.0, User Stories v1.0, FRS v1.0 |
| Date | 2026-09-16 |
| Purpose | Complete specification of the new responsive Farmer Web Portal and Farmer Login, including screen-by-screen behaviour and the farmer data-isolation rule |
| Source | 1. `AgriProcurement & Farmer Management.pdf` (primary source of truth) 2. `docs/00_DOCUMENTATION_INDEX.md` 3. `docs/01_PRODUCT_REQUIREMENTS_DOCUMENT.md` (§10) 4. `docs/02_BUSINESS_REQUIREMENTS_DOCUMENT.md` (§16–§17) 5. `docs/03_BUSINESS_WORKFLOWS.md` (W03–W05, W24–W28) 6. `docs/04_MODULE_SPECIFICATION.md` (M05) 7. `docs/05_USER_ROLES_AND_USER_STORIES.md` (US-043…US-052) 8. `docs/06_FUNCTIONAL_REQUIREMENTS.md` (FR-PRT-001…012) |

### Delivery principle

The original PDF states that **farmers will not have a separate mobile application in Phase 1** (`[SOURCE §1]`). This document does **not** change that into a mobile app. It introduces a **new responsive Farmer Web Portal** — used in a desktop or mobile web browser — with a **Farmer Login**. This is a **newly approved requirement (`[NEW]`)**, not part of the original PDF.

### Tagging conventions

| Tag | Meaning |
|---|---|
| `[SOURCE §n]` | Confirmed requirement — stated in the original PDF |
| `[NEW]` | Newly approved requirement — Farmer Portal decision |
| `[PROPOSED]` | Proposed future feature — not approved; shown for planning |

---

## 2. Requirement Separation

| Category | Scope |
|---|---|
| **Confirmed requirements (from source PDF)** | Farmer identity = permanent unique Farmer ID (`[SOURCE §3]`); registered mobile linked to Farmer ID for WhatsApp (`[SOURCE §3, §21]`); no native mobile app in Phase 1 (`[SOURCE §1]`); farmers receive WhatsApp purchase/payment/statement messages (`[SOURCE §8, §12, §13]`); Phase 2 WhatsApp chatbot (`[SOURCE §21]`). |
| **Newly approved requirements (`[NEW]`)** | Farmer Login; responsive Farmer Web Portal; portal screens (Dashboard, My Profile, My Purchases, My Invoices, My Payments, My Ledger, My Statements, Notifications, Support); OTP-based login via registered mobile — **Proposed Enhancement**, method pending (OQ-02); own-data-only rule and data isolation. |
| **Proposed future features (`[PROPOSED]`)** | See §16 — e.g., expanded support/ticketing, additional self-service capabilities, chatbot integration in portal UI. |

> No screen or behaviour below is delivered as a mobile application. The portal is browser-based and responsive (`[SOURCE §1]`, reaffirmed by `[NEW]`).

---

## 3. Farmer Portal Objective

Give every farmer a simple, secure, browser-based window into **their own** financial relationship with the business:

- Verify their own purchases, invoices, payments, ledger and statements.
- Access their own monthly statement PDFs without visiting the office.
- Reduce the burden on employees and admin of answering routine farmer queries.
- Lay a foundation (Phase 2+) for richer self-service without a mobile app.

The portal is **read-only for financial data** and is strictly scoped to the logged-in farmer's own records. It complements (does not replace) WhatsApp notifications (`[SOURCE §8, §12, §13]`).

Status: **`[NEW]`** — newly approved. Detailed behaviour of some screens remains undefined (OQ-01, OQ-16, OQ-17, OQ-18); where relevant this is stated.

---

## 4. Farmer Login

| Attribute | Detail |
|---|---|
| Purpose | Authenticate the farmer and open a session scoped to that farmer's own data. |
| Access | Responsive web page; also usable from a mobile browser. **Not a native mobile app.** |
| Actor | Farmer. |
| Trigger | Farmer opens the portal URL (`[NEW]`). |
| Identity used | Seed identity = the farmer's registered mobile number, which is linked to the Farmer ID (`[SOURCE §3]`). |

### Login flow

1. Farmer opens the portal login screen.
2. Farmer enters the registered mobile number (identity anchor per `[SOURCE §3]`).
3. System verifies a farmer exists for that mobile.
4. Farmer authenticates using the adopted method (see §5 — OTP is a **Proposed Enhancement**).
5. On success, a session is created with the farmer context; the farmer sees **only their own data**.

### Session behaviour
- Session timeout and management follow the system security baseline (`[SOURCE §18]`).
- Logout terminates the session.
- Session carries the Farmer ID; all data access is scoped server-side to that ID (see §6).

---

## 5. Authentication Requirements

| ID | Requirement | Status |
|---|---|---|
| P-AG-01 | The portal MUST require authentication before any farmer data is shown. | `[NEW]` |
| P-AG-02 | Authentication MUST be secure (per system security baseline `[SOURCE §18]`). | `[NEW]` / derived `[SOURCE §18]` |
| P-AG-03 | **OTP on the registered mobile number is a Proposed Enhancement** for login; the final authentication method is undefined (OQ-02). | `[PROPOSED]` |
| P-AG-04 | Rate limiting and session management MUST apply (per `[SOURCE §18]`). | `[NEW]` / derived `[SOURCE §18]` |
| P-AG-05 | Farmer sessions MUST be scoped to one Farmer ID; parallel/side access to another farmer's data MUST NOT be possible. | `[NEW]` (core rule, §6) |
| P-AG-06 | Password/credential policies for farmers, if credentials are used, are undefined (OQ-02). | Undefined |

---

## 6. Farmer Identity Mapping

| Concept | Relationship | Source |
|---|---|---|
| Farmer ID | Permanent, unique, primary business identity (e.g., F-0001). | `[SOURCE §3]` |
| Registered mobile | Linked to the Farmer ID; the identity anchor for WhatsApp and (portal) login seed. | `[SOURCE §3, §21]` |
| Farmer Master record | Contains Farmer ID, name, mobile, address, village/taluka/district/state, bank, KYC, status, products supplied. | `[SOURCE §3]` |

- **Farmer ID relationship:** The portal operates entirely in the context of the authenticated Farmer ID. Every data request (purchases, invoices, payments, ledger, statements, profile) is keyed to this ID.
- **Mobile number relationship:** The registered mobile identifies the farmer for WhatsApp (`[SOURCE §3]`) and is the proposed seed for portal login. The mobile → Farmer ID mapping must be resolved by the system, not by client-supplied input.
- Deduplication: a mobile number appearing against more than one Farmer ID is undefined (OQ-13) and must be resolved before portal identity is trustworthy.

---

## 7. The Critical Rule — A Farmer Can Access ONLY Their Own Data

This is both a **business rule** and an **authorization rule**.

### 7.1 Business requirement

Each farmer is entitled to see their own business relationship: their profile, purchases, invoices, payments, ledger and statements. No farmer is entitled to see, copy or infer the records of any other farmer. This protects farm-level privacy and financial confidentiality and maintains trust in the business relationship.

### 7.2 Authorization requirement

- The system MUST enforce data access scope on the **backend / database authorization level** (derived from `[SOURCE §22]` where employee area restriction is likewise backend-enforced; the same server-side enforcement principle applies to farmers).
- The logged-in Farmer ID is the **only** permitted data key for portal queries.
- The farmer MUST NOT be able to access another farmer's:
  - **Profile**
  - **Purchases**
  - **Invoices**
  - **Payments**
  - **Ledger**
  - **Statements**

### 7.3 Enforcement requirements

| ID | Requirement | Status |
|---|---|---|
| P-ISO-01 | Every portal data request MUST be scoped server-side to the authenticated Farmer ID. Farmer-supplied identifiers (e.g., the farmer IDs, invoice IDs in URLs/requests) MUST NOT be relied on alone; they must be validated to belong to the session farmer. | `[NEW]` |
| P-ISO-02 | The portal MUST NOT expose direct record access patterns that bypass the farmer scope (e.g., object enumeration). | `[NEW]` |
| P-ISO-03 | Attempts to access other farmers' data MUST be denied, not silently redirected. | `[NEW]` |
| P-ISO-04 | Denied cross-scope access attempts MUST be logged (security/audit, per `[SOURCE §18]` auditing intent). | `[NEW]` / `[PROPOSED]` detail |
| P-ISO-05 | Search/global lookup capabilities MUST NOT be present in the farmer portal (farmer sees only pre-scoped lists). | `[NEW]` |
| P-ISO-06 | Sensitive data display (bank details, KYC) is limited to the farmer's own profile and view/edit scope pending OQ-18. | `[NEW]` |
| P-ISO-07 | Server-side session references Farmer ID; logout/expiry MUST invalidate access. | `[NEW]` |

---

## 8. Screens — Common Specification

The following attributes are repeated for every screen; the table below defines the common behaviour, and each screen section then states only its specifics.

| Attribute | Common definition |
|---|---|
| Purpose | What the screen achieves for the farmer. |
| Information displayed | Data shown (always scoped to the logged-in farmer). |
| Actions | What the farmer can do on the screen. |
| Permissions | Farmer — own data only (per §7). |
| Data required | Underlying records. |
| Validation | Client-side/business checks relevant to the screen. |
| Empty state | Message/behaviour when no data exists. |
| Error state | Message/behaviour when the screen cannot be loaded. |
| Loading state | Transient loading indicator while data is fetched. |
| Security considerations | Isolation, sensitive data, and request-scoping notes. |

**Common rules for all screens**
- Every data call is scoped to the session Farmer ID (P-ISO-01).
- Screens are **read-only** for financial data unless otherwise stated.
- Loading, empty and error states MUST be handled gracefully on a responsive layout.
- No screen allows browsing/searching data outside the farmer's own scope (P-ISO-05).
- Bank and KYC data display rules pending OQ-18 (P-ISO-06).

---

## 9. Screen: Dashboard

| Attribute | Detail |
|---|---|
| Purpose | At-a-glance summary of the farmer's own position. |
| Information displayed | Own summary, e.g., recent purchases, recent payments, outstanding balance, latest statement period. (Exact widgets undefined — OQ-01.) |
| Actions | Navigate to other portal sections (My Profile, My Purchases, My Invoices, My Payments, My Ledger, My Statements, Notifications, Support). |
| Permissions | Farmer — own data. |
| Data required | Aggregated own purchase/payment/invoice/ledger data; outstanding. |
| Validation | Figures drawn from system of record; no free-text input. |
| Empty state | Farmer with no transactions → friendly no-activity message (detail undefined). |
| Error state | Data load failure → retry message; no partial other-farmer data ever shown (P-ISO-01/02). |
| Loading state | Skeleton/spinner while summaries load. |
| Security considerations | Aggregates computed server-side for session Farmer ID; never for others. |

Status: **`[NEW]`** (N-02). Widgets undefined — OQ-01.

---

## 10. Screen: My Profile

| Attribute | Detail |
|---|---|
| Purpose | Let the farmer view (and, subject to OQ-18, maintain) their own master data. |
| Information displayed | Own Farmer ID, name, mobile, address, village/taluka/district/state, bank name/account/IFSC, KYC status, registration date, status. |
| Actions | View details; edit supported fields **if business approves** (edit scope undefined — OQ-18); update mobile/KYC via defined process (undefined). |
| Permissions | Farmer — own profile only (owned by the shown Farmer ID). |
| Data required | Farmer Master record; bank; KYC status. |
| Validation | Format checks for mobile/IFSC if editing is enabled (pending OQ-18). |
| Empty state | N/A (profile always exists). |
| Error state | Sensitive data (bank/KYC) masked or restricted per policy; load errors surfaced. |
| Loading state | Spinner while profile loads. |
| Security considerations | Bank details are sensitive (`[SOURCE §18]`); access restricted to self; display rules pending OQ-18 (P-ISO-06). |

Status: **`[NEW]`**. Edit scope undefined — OQ-18.

---

## 11. Screen: My Purchases

| Attribute | Detail |
|---|---|
| Purpose | Let the farmer see everything bought from them. |
| Information displayed | Own purchase transactions: date/time, product, quantity/weight, unit, rate, gross amount, deduction (if any), net amount, remarks, invoice reference. |
| Actions | View list/details; navigate to the related invoice (My Invoices). |
| Permissions | Farmer — own purchases only (P-ISO-01). |
| Data required | Procurement transaction records for the session Farmer ID. |
| Validation | List pagination/filters if provided (undefined); no data mutation. |
| Empty state | "No purchases recorded yet" message (behaviour undefined in source). |
| Error state | Load failure → retry; never other farmers' data. |
| Loading state | Spinner while list loads. |
| Security considerations | Records keyed to session farmer; cannot access other farmers' purchases (P-ISO-01/05). |

Status: **`[NEW]`**.

---

## 12. Screen: My Invoices

| Attribute | Detail |
|---|---|
| Purpose | Let the farmer see their own invoiced sales. |
| Information displayed | Own invoices: invoice number, date, product, quantity, rate, total, status, employee reference (per `[SOURCE §7]` confirmation fields). |
| Actions | View invoice details; open a printable/PDF version **if approved** (invoice output undefined — OQ-07); navigate to payments. |
| Permissions | Farmer — own invoices only. |
| Data required | Invoice records for the session Farmer ID. |
| Validation | None beyond scope check. |
| Empty state | "No invoices" message. |
| Error state | Load failure → retry; no cross-farmer leak. |
| Loading state | Spinner while list loads. |
| Security considerations | Invoice numbers are unique per farmer (`[SOURCE §6]`); access validated against session farmer (P-ISO-01). |

Status: **`[NEW]`**. Printable/PDF invoice pending OQ-07.

---

## 13. Screen: My Payments

| Attribute | Detail |
|---|---|
| Purpose | Let the farmer see payments made to them. |
| Information displayed | Own payments: payment date, amount, payment mode, bank reference, UTR, status, allocation to invoices. |
| Actions | View payment details; navigate to related invoices. |
| Permissions | Farmer — own payments only. |
| Data required | Payment records + allocations for the session Farmer ID. |
| Validation | None beyond scope check. |
| Empty state | "No payments yet" message. |
| Error state | Load failure → retry; no cross-farmer leak. |
| Loading state | Spinner while list loads. |
| Security considerations | UTR is a bank reference — sensitive but shown to the legitimate farmer (the payee). Only the session farmer's UTRs are shown (P-ISO-01). |

Status: **`[NEW]`**.

---

## 14. Screen: My Ledger

| Attribute | Detail |
|---|---|
| Purpose | Let the farmer see their own financial ledger and standing. |
| Information displayed | Own ledger per `[SOURCE §9]`: date, invoice, product, quantity, rate, amount, payment, UTR; outstanding balance. |
| Actions | View entries; filter by period **if provided** (undefined); navigate to invoices/payments. |
| Permissions | Farmer — own ledger only. |
| Data required | Ledger entries for the session Farmer ID. |
| Validation | None beyond scope check. |
| Empty state | "No ledger entries" message. |
| Error state | Load failure → retry; no cross-farmer leak. |
| Loading state | Spinner while ledger loads. |
| Security considerations | Ledger is financial; strictly farmer-scoped (P-ISO-01). No silent-overwrite history exposed beyond what the system of record holds (`[SOURCE §17]`). |

Status: **`[NEW]`** (content per `[SOURCE §9]`).

---

## 15. Screen: My Statements

| Attribute | Detail |
|---|---|
| Purpose | Let the farmer view and download their own monthly statements. |
| Information displayed | Own statements by period (same PDF artifact delivered via WhatsApp, `[SOURCE §13]`): period, opening balance, purchases, payments with dates/UTRs, closing/outstanding. |
| Actions | View list by period; download/view PDF; navigate to ledger. |
| Permissions | Farmer — own statements only. |
| Data required | Statement artifacts for the session Farmer ID. |
| Validation | Statement must belong to the session farmer (P-ISO-01). |
| Empty state | "No statement available for this period yet" (increase consistent with `[SOURCE §13]` generation cycle). |
| Error state | Load/download failure → retry; no cross-farmer leak. |
| Loading state | Spinner while statement list/PDF loads. |
| Security considerations | PDF access scoped to session farmer; a statement's Farmer ID must equal the session farmer's ID before serving (P-ISO-01/03). |

Status: **`[NEW]`** (statement generation per `[SOURCE §13]`).

---

## 16. Screen: Notifications

| Attribute | Detail |
|---|---|
| Purpose | Let the farmer revisit messages sent to them (purchase, payment, statement). |
| Information displayed | Own notification history (what was sent to the farmer's WhatsApp, per `[SOURCE §8, §12, §13]`), e.g., type, date, summary, delivery status if tracked. |
| Actions | View notifications; link to related invoice/payment/statement. |
| Permissions | Farmer — own notifications only. |
| Data required | Notification/delivery records for the session Farmer ID (full status tracking is a Proposed Enhancement — OQ-04 related). |
| Validation | Scope check only. |
| Empty state | "No notifications" message. |
| Error state | Load failure → retry. |
| Loading state | Spinner while list loads. |
| Security considerations | Notification content is farmer-specific; scoped to session (P-ISO-01). |

Status: **`[NEW]`**; earliest record / retention undefined.

---

## 17. Screen: Support

| Attribute | Detail |
|---|---|
| Purpose | Give the farmer a way to reach help. |
| Information displayed | Support contact channel(s)/guidance. Channel undefined — OQ-16. |
| Actions | Contact support (channel per business decision). |
| Permissions | Farmer (authenticated). |
| Data required | Contact info; optionally farmer context for the request (undefined). |
| Validation | Channel-dependent (undefined). |
| Empty state | N/A. |
| Error state | Channel unavailable → informational message. |
| Loading state | N/A (static guidance). |
| Security considerations | Support requests must not leak other farmers' data; identity verified by authenticated session. |

Status: **`[NEW]`**; channel undefined — OQ-16.

---

## 18. Future Farmer Features (`[PROPOSED]`)

These are proposals for later phases. **None are approved.** None introduce a mobile application — all remain browser-based in the responsive portal consistent with `[SOURCE §1]`.

| # | Feature | Description | Status |
|---|---|---|---|
| P-FF-01 | WhatsApp Chatbot in portal UI | Surface the Phase 2 chatbot queries (`[SOURCE §21]`) as an in-portal Q&A instead of (or alongside) WhatsApp. Depends on Phase 2 chatbot. | `[PROPOSED]` |
| P-FF-02 | Advanced portal notifications | Delivery-status tracking and read/unread within the portal. | `[PROPOSED]` |
| P-FF-03 | Profile self-service | Farmer-editable contact fields with approval workflow. | `[PROPOSED]` |
| P-FF-04 | Support ticketing | Formal ticket + status in the portal. | `[PROPOSED]` |
| P-FF-05 | Language and formatting preferences | Portal language(s) per OQ-17. | `[PROPOSED]` |
| P-FF-06 | Portal-driven account activation | Self-service onboarding/activation flow per OQ-15. | `[PROPOSED]` |
| P-FF-07 | Document centre | Central access to invoices and statements beyond the current modules. | `[PROPOSED]` |

---

## 19. Alignment with Existing Documentation

| This spec | Other documents |
|---|---|
| Farmer Login / authentication | PRD §10 (PRD-PRT-001…); BRD §16; Workflows W03/W04; FR-PRT-001/002 |
| Dashboard / view screens | Workflows W05, W24–W28; FR-PRT-003…011; US-044…052 |
| Own-data-only rule | FR-PRT-012, FR-PRT-016 (PRD); BRD §16.4; derived from `[SOURCE §22]` enforcement principle |
| No mobile app | `[SOURCE §1]`; PRD §6; N-03; BRD §16.2 |

---

## 20. Open Questions for the Farmer Portal

| Ref | Question | Status |
|---|---|---|
| OQ-01 | Portal delivery phase and exact widget/field definitions | `[NEW]` |
| OQ-02 | Farmer authentication method (OTP proposed) | `[NEW]` |
| OQ-13 | Same mobile against two Farmer IDs (identity trust) | Undefined |
| OQ-15 | Portal account activation / onboarding | `[NEW]` |
| OQ-16 | Support channel definition | `[NEW]` |
| OQ-17 | Portal languages / display formats | `[NEW]` |
| OQ-18 | What farmers may view/edit in profile (bank, KYC) | `[NEW]` |
| OQ-07 | Printable/PDF invoices in portal | Undefined |

---

*End of Farmer Portal Specification v1.0. Next in sequence: `08_API_SPECIFICATION.md` (or per index reading order).*