# Agri Procurement & Farmer Management System
## REST API Specification

---

## 1. Document Information

| Item | Value |
|---|---|
| Product name | Agri Procurement & Farmer Management System |
| Document | REST API Specification |
| Version | v1.0 |
| Status | Draft — aligned to PRD v1.0, FRS v1.0, Modules v1.0, Portal Spec v1.0, Data Model v1.0, RBAC v1.0, Audit v1.0 |
| Date | 2026-09-16 |
| Author role | Senior Backend Architect |
| Purpose | Complete REST API specification for all capabilities, grouped by domain, with versioning, authentication, authorization, validation, responses, HTTP status codes and audit |
| Source | 1. `docs/01_PRODUCT_REQUIREMENTS_DOCUMENT.md` 2. `docs/06_FUNCTIONAL_REQUIREMENTS.md` 3. `docs/07_FARMER_PORTAL_SPECIFICATION.md` 4. `docs/08_…09_…10_…11_PROCUREMENT/PAYMENT/LEDGER/WHATSAPP specs` 5. `docs/12_RBAC_AND_AUTHORIZATION.md` 6. `docs/14_AUDIT_LOGGING_AND_DATA_INTEGRITY.md` 7. `docs/15_DATABASE_DESIGN.md` |

### Conventions

| Tag | Meaning |
|---|---|
| `[SOURCE §n]` | Requirement in original PDF section `n` |
| `[NEW]` | Newly approved Farmer Portal requirement |
| `[PROPOSED]` | Technical design suggestion; not a source requirement |
| Phase 2 | Area/chatbot capabilities |
| Undefined | Detail not specified in the source requirements; listed in §20 |

### Ground rules

1. No backend implementation code is produced in this document — only API contracts.
2. Versioned namespace `/api/v1/…` is used throughout; later versions use `/api/v2/…` when breaking changes are required.
3. **Farmer (portal) endpoints derive identity from the authenticated session** (`/farmer/me/…`); arbitrary farmer IDs from the client are never used as the data-scope key (see §4).
4. All data-layer scoping follows DBZ-01/02/03 and TC-05 (`docs/15`): farmer = session Farmer ID; employee (Phase 2) = assigned area.
5. Provider-neutral: banking and WhatsApp external/provider callback endpoints are placeholders with no vendor-specific payloads (`[SOURCE §11]`; Q-PAY-001, Q-WH-01).

---

## 2. Versioning & Base

| Aspect | Value |
|---|---|
| Base path | `/api/v1` (illustrative host: `https://portal.example.com`) |
| Versioning | Path versioning; v1 initial. Breaking changes require a new major version. |
| Content type | `application/json` (PDF/download: `application/pdf` + `Content-Disposition`) |
| Date/time | ISO 8601 (timezone policy undefined — Q-DB-09) |
| Money | Decimal values carried as strings to avoid floating-point loss (scale per Q-DB-02) |
| Pagination | Query `page` (1-based), `size` (default 20, max 100); response envelope includes `page`, `size`, `total` |
| Idempotency | Mutating payment/allocations accept optional `Idempotency-Key` header to prevent double-submission (`[PROPOSED]`, guards Q-PAY-006) |

---

## 3. Global Conventions

### 3.1 Authentication

- All endpoints except explicitly marked **Public** require a valid session token: `Authorization: Bearer <token>`.
- Token obtained via `POST /auth/login` (or farmer flow §5). Secure API authentication is mandatory (`[SOURCE §18]`).
- Sessions expire and are managed per PRD-SEC-011; rate limiting per FR-AUTH-006.
- Provider webhooks are authenticated by provider-specific signing after vendor selection (placeholder; not specified — Q-PAY-001/Q-WH-01).

### 3.2 Authorization

- **Required role** = minimum role for the call; evaluated against the configurable permission matrix (`[SOURCE §16]`).
- **Authorization rule** = the data-scope check applied server-side in addition to role:
  - Farmer calls: scope = session Farmer ID (F-OD, P-ISO-01; `docs/12` §7).
  - Employee calls: Phase 1 procurement-scope farmer info (`[SOURCE §16]`); Phase 2 assigned-area scope (E-AR-01/02, DBZ-02/03).
  - Admin: full scope (`[SOURCE §2]`).
- Denied calls return 403 (or 404 where a resource must not be revealed) and ARE logged (§16; P-ISO-04).

### 3.3 Standard responses

Success: `200 OK` (get/update), `201 Created` (create), `202 Accepted` (async initiation), `204 No Content` (delete/suppress).

Error envelope (all 4xx/5xx):

```
{ "error": { "code": "...", "message": "...", "fieldErrors": [ { "field": "...", "message": "..." } ], "traceId": "..." } }
```

| HTTP status | Meaning |
|---|---|
| 400 | Validation failure — request malformed or violates business rules |
| 401 | Unauthenticated / invalid or expired token |
| 403 | Authenticated but not permitted (role/data-scope) |
| 404 | Resource not found (also used to avoid disclosing existence of non-owned records) |
| 409 | State conflict (e.g., duplicate payment, invoice already cancelled, allocation over-due) |
| 429 | Rate limit exceeded |
| 500 | Internal error |

### 3.4 Audit requirements (applies to every endpoint)

- Every **mutating** API (POST/PUT/PATCH/DELETE) writes an audit entry: actor type+ID, date/time, action, entity, record reference, original value, new value, IP, device (`[SOURCE §17]`; `docs/14` §4, E16).
- **Login/logout, permission changes, invoice changes/cancellation, payment/reconciliation changes, config changes** carry mandatory audit (`[SOURCE §6, §17]`).
- Denied cross-scope attempts are logged (P-ISO-04).
- Read endpoints are audited only where required (reports/admin logs; not specified for ordinary reads).
- Where an endpoint says **Audit:** `§17 (mandatory)` it always writes; otherwise audit is situationally defined.

Endpoints below state only audit highlights; §3.4 is the baseline.

---

## 4. Identity-Scoped Farmer API Design (Critical)

**Requirement:** "Farmer APIs should prefer authenticated identity rather than trusting arbitrary farmer IDs from the client."

**Approach adopted: `GET /api/v1/farmer/me/...` for all farmer-scoped data.**

### Design decision — self-scoped endpoints vs arbitrary-ID endpoints

| Pattern | Farmer portal | Admin/employee |
|---|---|---|
| Farmer self-service reads | `GET /api/v1/farmer/me/ledger` (identity from token) | — |
| Admin/employee farmer-ledger reads | — | `GET /api/v1/ledger/farmers/{farmerId}` (SUPER_ADMIN; employee per matrix) |
| **Unrestricted** `GET /api/v1/farmers/:farmerId/ledger` (any farmer can pass any ID) | **NOT exposed** | **NOT exposed** |

### Why (documented)

1. **Server-owned scope.** The session's Farmer ID is the only data key (P-ISO-01; `docs/07` §7.3). Client-supplied farmer IDs are "validated to belong to the session farmer," never trusted alone.
2. **No enumeration surface.** Arbitrary-ID endpoints invite loan-object enumeration and tampering (P-ISO-02). `/farmer/me/…` removes the attack surface entirely.
3. **Denial semantics.** A request to another farmer's data must be denied, not silently redirected (P-ISO-03); self-scoped endpoints make cross-farmer access structurally impossible.
4. **Deduplication safety.** Identity ambiguity (same mobile → two Farmer IDs, OQ-13) is resolved server-side at login; the session binds one Farmer ID, so portal queries never re-derive owner from payload.
5. **Enforcement at the data layer.** The pattern maps directly to DBZ-01/TC-05 scoped predicates; role/matrix and data scope stay server-side (BZ-01).

**Consequence:** the farmer portal has **no** `GET /farmers/{farmerId}/…` family at all. Where the business legitimately needs an admin's farmer-perspective view, it is under separate admin-scoped resources (e.g., `GET /ledger/farmers/{farmerId}`, `GET /invoices?farmerId=`).

---

## 5. Group 1 — Authentication

Common: all endpoints **Public** except `/auth/me`; rate-limited (FR-AUTH-006); login events audited.

### 5.1 POST /auth/login — staff login
- **Method/Endpoint:** `POST /api/v1/auth/login`
- **Purpose:** Authenticate Admin/Employee and obtain a session token (`[SOURCE §18]`).
- **Authentication:** Public (no token).
- **Required role:** None.
- **Authorization rule:** n/a — resolves account status (blocked accounts denied).
- **Path parameters:** none.
- **Query parameters:** none.
- **Request body:** `{ "loginId": string, "password": string, "otp": string? }` (OTP/2FA `[SOURCE §18]`).
- **Validation:** account exists; active; credentials match; policy/2FA per FR-AUTH-002/003; strong-password parameters undefined.
- **Success response:** `200` `{ "accessToken": "...", "tokenType": "Bearer", "expiresIn": seconds, "user": { "userId", "role", "employeeId" } }`
- **Error response:** `401` invalid/missing credentials/2FA; `403` blocked/disabled account; `429` rate limit; standard envelope.
- **HTTP status codes:** 200, 401, 403, 429.
- **Audit:** §17 (mandatory) — login success/failure incl. IP/device (FR-AUTH-001).

### 5.2 POST /auth/refresh
- **Method/Endpoint:** `POST /api/v1/auth/refresh`
- **Purpose:** Renew an expiring session token (PRD-SEC-011).
- **Authentication:** Bearer (current token).
- **Required role:** Any authenticated user.
- **Authorization rule:** token still valid/not revoked.
- **Path/Query:** none.
- **Request body:** `{ "refreshToken": "..." }` (`[PROPOSED]` token model).
- **Validation:** token issued for same principal; not revoked.
- **Success response:** `200` new token payload.
- **Error response:** `401` expired/revoked.
- **HTTP status codes:** 200, 401, 429.
- **Audit:** §17 — session reissue.

### 5.3 POST /auth/logout
- **Method/Endpoint:** `POST /api/v1/auth/logout`
- **Purpose:** Terminate the session (FR-AUTH-005).
- **Authentication:** Bearer.
- **Required role:** Any authenticated user.
- **Authorization rule:** own session.
- **Body:** none.
- **Validation:** token present.
- **Success response:** `204`.
- **Error response:** `401` if token invalid.
- **HTTP status codes:** 204, 401.
- **Audit:** §17 — logout/session termination.

### 5.4 POST /auth/farmer/otp/request — farmer OTP (`[PROPOSED]`, definition OQ-02)
- **Method/Endpoint:** `POST /api/v1/auth/farmer/otp/request`
- **Purpose:** Request a one-time code to the registered mobile (Proposed Enhancement; final farmer auth method pending OQ-02).
- **Authentication:** Public; rate-limited.
- **Required role:** None.
- **Authorization rule:** n/a.
- **Request body:** `{ "mobile": string }`
- **Validation:** mobile is a registered farmer mobile (system resolves Farmer ID `[SOURCE §3]`); duplicate-mobile ambiguity per OQ-13.
- **Success response:** `202` `{ "requestId": "...", "expiresIn": seconds }` (no code echoed).
- **Error response:** `404` mobile not registered (non-disclosing); `429` limits.
- **HTTP status codes:** 202, 404, 429.
- **Audit:** §17 — OTP request/failure events (Proposed).

### 5.5 POST /auth/farmer/otp/verify
- **Method/Endpoint:** `POST /api/v1/auth/farmer/otp/verify`
- **Purpose:** Verify code, create farmer session bound to one Farmer ID (FR-PRT-001, FR-AUTH-007).
- **Authentication:** Public; rate-limited.
- **Required role:** None.
- **Authorization rule:** n/a — session binds resolved Farmer ID; **subsequent calls rely on this identity, not client IDs** (§4).
- **Request body:** `{ "requestId": "...", "otp": "..." }`
- **Validation:** code matches; within expiry; attempts limited.
- **Success response:** `200` farmer session token (+ `farmerId`, masked).
- **Error response:** `401` invalid/expired OTP; `429`.
- **HTTP status codes:** 200, 401, 429.
- **Audit:** §17 — farmer login events (Proposed, consistent `[SOURCE §18]`).

### 5.6 GET /auth/me
- **Method/Endpoint:** `GET /api/v1/auth/me`
- **Purpose:** Return current principal (role, IDs) for session bootstrap.
- **Authentication:** Bearer.
- **Required role:** Any authenticated.
- **Authorization rule:** own principal.
- **Success response:** `200` user/role/employee/farmer context.
- **Error response:** `401` invalid session.
- **Audit:** none (session verification).

---

## 6. Group 2 — Admin

### 6.1 GET /admin/dashboard/kpis
- **Method/Endpoint:** `GET /api/v1/admin/dashboard/kpis`
- **Purpose:** Admin KPI dashboard (`[SOURCE §15]`): total farmers, active farmers, total employees, today's procurement, today's procurement value, monthly procurement, pending farmer payments, payments made, total outstanding, number of invoices, unreconciled bank transactions.
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Authorization rule:** full scope (`[SOURCE §2]`).
- **Path:** none.
- **Query:** `from`, `to` (dates; dashboard date filters `[SOURCE §15]`).
- **Body:** none.
- **Validation:** date range valid.
- **Success response:** `200` KPI object.
- **Error:** 401/403; `400` bad range.
- **HTTP status codes:** 200, 400, 401, 403.
- **Audit:** §17 (privileged metric access where required).

### 6.2 GET /admin/permission-matrix
- **Method/Endpoint:** `GET /api/v1/admin/permission-matrix`
- **Purpose:** Read the configurable permission matrix (`[SOURCE §16]`).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Authorization rule:** full scope.
- **Success response:** `200` matrix (role × module × action × scope).
- **Error:** 401/403.
- **HTTP status codes:** 200, 401, 403.
- **Audit:** none for read.

### 6.3 PUT /admin/permission-matrix
- **Method/Endpoint:** `PUT /api/v1/admin/permission-matrix`
- **Purpose:** Update permission matrix (`[SOURCE §16]`).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Authorization rule:** full scope.
- **Request body:** matrix revision payload.
- **Validation:** roles/permission codes exist; valid transitions.
- **Success:** `200` updated matrix.
- **Error:** `400` invalid entries; `409` concurrent edit.
- **HTTP status codes:** 200, 400, 401, 403, 409.
- **Audit:** §17 (mandatory) — matrix change original/new (FR-AUTH-004).

### 6.4 (Phase 2) Areas
- `GET /api/v1/admin/areas` — list areas (`[SOURCE §23]`); `POST /api/v1/admin/areas` — create; `PUT /api/v1/admin/areas/{areaId}` — update. All SUPER_ADMIN; audit §17 mandatory; body/validation per Area master (E18). Phase 2.

---

## 7. Group 3 — Employee

### 7.1 GET /employees
- **Method/Endpoint:** `GET /api/v1/employees`
- **Purpose:** List employees (filters/pagination).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Authorization rule:** full scope.
- **Query:** `page`, `size`, `status`, `search`.
- **Success:** `200` paged employee list (sensitive fields omitted).
- **Error:** 401/403.
- **Audit:** none for read.

### 7.2 POST /employees
- **Method/Endpoint:** `POST /api/v1/employees`
- **Purpose:** Create employee master + account (`[SOURCE §4]`).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Authorization rule:** full scope.
- **Request body:** name, mobile, email, credentials, role, status.
- **Validation:** required fields per `[SOURCE §4]`; role/status valid (OQ-09); unique employee ID generation (FR-EMP-002).
- **Success:** `201` employee record (no credential echo).
- **Error:** `400` validation; `409` duplicate login/mobile.
- **HTTP status codes:** 201, 400, 401, 403, 409.
- **Audit:** §17 (mandatory).

### 7.3 PUT /employees/{employeeId}
- **Method/Endpoint:** `PUT /api/v1/employees/{employeeId}`
- **Purpose:** Update employee master/role/status/credentials (FR-EMP-003).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Authorization rule:** full scope.
- **Path:** `employeeId` (e.g., EMP-001).
- **Body:** subset updates.
- **Validation:** role/status valid; credential policy FR-AUTH-002.
- **Success:** `200`. **Error:** 400/404/409.
- **Audit:** §17 (mandatory) — credential/role/status changes (FR-EMP-003).

### 7.4 (Phase 2) Area assignment
- `POST /api/v1/employees/{employeeId}/area` — assign/transfer employee to area (`[SOURCE §23]`; M22). SUPER_ADMIN; audit §17 mandatory. Phase 2.

---

## 8. Group 4 — Farmer (Farmer Master, admin-managed)

### 8.1 GET /farmers/search (employee procurement lookup)
- **Method/Endpoint:** `GET /api/v1/farmers/search?q=`
- **Purpose:** Employee searches farmer to start procurement (FR-FRM-007, `[SOURCE §5]`). **Employee reads only procurement-scope info** (`[SOURCE §16]`).
- **Authentication:** Bearer. **Required role:** EMPLOYEE (or SUPER_ADMIN).
- **Authorization rule:** Employee — **Phase 1:** names/mobile/ID needed for procurement only (`[SOURCE §16]`); **Phase 2:** restricted to assigned area (E-AR-01/02; DBZ-02/03). Never exposes bank/KYC.
- **Path:** none. **Query:** `q` (ID/name/mobile), `page`, `size`.
- **Validation:** query non-empty.
- **Success:** `200` paged farmer matches (ID, name, village/taluka/district, status) — no bank/KYC fields.
- **Error:** 400/401/403; `404` (no match, non-disclosing).
- **HTTP status codes:** 200, 400, 401, 403.
- **Audit:** §17 (lookup logged as applicable FR-FRM-007).

### 8.2 POST /farmers
- **Method/Endpoint:** `POST /api/v1/farmers`
- **Purpose:** Create Farmer Master with automatic permanent Farmer ID (FR-FRM-001/002).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Authorization rule:** full scope.
- **Request body:** name, mobile, address, village/taluka/district/state, bank (name/account/IFSC), KYC where required, registration date, status, products normally supplied, remarks (`[SOURCE §3]`).
- **Validation:** required fields; mobile format; IFSC/account formats (FR-FRM-005); sensitive fields encrypted/restricted (`[SOURCE §18]`).
- **Success:** `201` farmer record with `farmerId` (sensitive display masked).
- **Error:** 400; 409 (OQ-13 mobile ambiguity where policy applies).
- **Audit:** §17 (mandatory).

### 8.3 GET /farmers/{farmerId}
- **Method/Endpoint:** `GET /api/v1/farmers/{farmerId}`
- **Purpose:** View farmer master (admin).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN (matrix-dependent for employees, OQ-09).
- **Authorization rule:** admin full; employee Phase 2 area-scoped (E-AR-01/02).
- **Path:** `farmerId` (F-0001).
- **Success:** `200` farmer record (bank/KYC masked per role `[SOURCE §18]`).
- **Error:** 401/403/404.
- **Audit:** none for read.

### 8.4 PUT /farmers/{farmerId}
- **Method/Endpoint:** `PUT /api/v1/farmers/{farmerId}`
- **Purpose:** Update farmer master (FR-FRM-006).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Authorization rule:** full scope.
- **Body:** updateable fields.
- **Validation:** status values/transitions (OQ); sensitive fields restricted.
- **Success:** `200`. **Error:** 400/404/409.
- **HTTP status codes:** 200, 400, 401, 403, 404.
- **Audit:** §17 (mandatory); farmer status/block changes included (FR-FRM-006).

### 8.5 POST /farmers/{farmerId}/kyc
- **Method/Endpoint:** `POST /api/v1/farmers/{farmerId}/kyc`
- **Purpose:** Capture KYC documents (FR-FRM-004, `[SOURCE §3]`).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Authorization rule:** full scope.
- **Body:** document_type, document_ref, document file (multipart; stored as ref).
- **Validation:** KYC policy undefined (OQ-10).
- **Success:** `201` KYC record.
- **Error:** 400/404.
- **Audit:** §17 (mandatory).

### 8.6 GET /farmers/{farmerId}/kyc
- **Method/Endpoint:** `GET /api/v1/farmers/{farmerId}/kyc`
- **Purpose:** List farmer KYC (admin; restricted access `[SOURCE §18]`).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Authorization rule:** full scope; sensitive-token only.
- **Success:** `200` list (document contents restricted).
- **Error:** 403/404.
- **Audit:** none for read.

### 8.7 (Phase 2) Farmer area assignment
- `POST /api/v1/farmers/{farmerId}/area` — assign/reassign farmer to area (`[SOURCE §23]`; M21). SUPER_ADMIN; audit §17 mandatory; reassignment history audited. Phase 2.

---

## 9. Group 5 — Farmer Portal (`[NEW]`) — identity-scoped

> All endpoints in this group resolve data **from the authenticated farmer session**. There is **no** `farmerId` path parameter, and no ownership is inferred from client input (§4, P-ISO-01).

### 9.1 GET /farmer/me/dashboard
- **Method/Endpoint:** `GET /api/v1/farmer/me/dashboard`
- **Purpose:** Own summary — recent purchases, recent payments, outstanding, latest statement (FR-PRT-003; widgets OQ-01).
- **Authentication:** Bearer (farmer session). **Required role:** FARMER.
- **Authorization rule:** scope = session Farmer ID (F-OD, P-ISO-01). Aggregates computed server-side for session farmer only.
- **Path/Query:** none.
- **Success:** `200` own dashboard object.
- **Error:** 401/403/404 (non-disclosing).
- **HTTP status codes:** 200, 401, 403.
- **Audit:** none (view).

### 9.2 GET /farmer/me/profile
- **Method/Endpoint:** `GET /api/v1/farmer/me/profile`
- **Purpose:** Own profile — identity, contact, village/taluka/district/state, bank, KYC status (FR-PRT-004).
- **Authentication:** Bearer. **Required role:** FARMER.
- **Authorization rule:** session owner only (P-ISO-06; display rules OQ-18).
- **Success:** `200` own profile (bank/KYC masked per policy OQ-18).
- **Error:** 401/403.
- **Audit:** none (view).

### 9.3 GET /farmer/me/purchases
- **Method/Endpoint:** `GET /api/v1/farmer/me/purchases`
- **Purpose:** Own purchases (FR-PRT-005): date/time, product, quantity/unit, rate, gross, deduction, net, remarks, invoice ref.
- **Authentication:** Bearer. **Required role:** FARMER.
- **Authorization rule:** session scope (P-ISO-01/05).
- **Query:** `page`, `size`, `from`, `to` (period filters if provided — undefined).
- **Success:** `200` paged own purchases.
- **Error:** 401/403.
- **Audit:** none (view); denial logging per P-ISO-04.

### 9.4 GET /farmer/me/invoices
- **Method/Endpoint:** `GET /api/v1/farmer/me/invoices`
- **Purpose:** Own invoices + status (FR-PRT-006).
- **Authentication:** Bearer. **Required role:** FARMER.
- **Authorization rule:** session scope.
- **Query:** `page`, `size`.
- **Success:** `200` paged own invoices.
- **Error:** 401/403.
- **Audit:** none (view).

### 9.5 GET /farmer/me/invoices/{invoiceNumber}
- **Method/Endpoint:** `GET /api/v1/farmer/me/invoices/{invoiceNumber}`
- **Purpose:** Single own invoice detail (e.g., F-0001-17).
- **Authentication:** Bearer. **Required role:** FARMER.
- **Authorization rule:** invoice's Farmer ID MUST equal session Farmer ID; otherwise `404` (P-ISO-01/03) — the endpoint validates ownership against the session, never trusts the number alone.
- **Path:** `invoiceNumber`.
- **Success:** `200` invoice detail per `[SOURCE §7]` fields.
- **Error:** 401/403/404 (non-disclosing).
- **Audit:** denied cross-scope attempts logged (P-ISO-04).

### 9.6 GET /farmer/me/payments
- **Method/Endpoint:** `GET /api/v1/farmer/me/payments`
- **Purpose:** Own payments (FR-PRT-007): date, amount, mode, bank ref, UTR, status, allocation.
- **Authentication:** Bearer. **Required role:** FARMER.
- **Authorization rule:** session scope.
- **Query:** `page`, `size`.
- **Success:** `200` paged own payments (UTRs of the payee shown — `docs/07` §13).
- **Error:** 401/403.
- **Audit:** none (view).

### 9.7 GET /farmer/me/ledger
- **Method/Endpoint:** `GET /api/v1/farmer/me/ledger`
- **Purpose:** Own ledger per `[SOURCE §9]` + outstanding (FR-PRT-008). **The prescribed example: authenticated identity, no farmer ID from client.**
- **Authentication:** Bearer. **Required role:** FARMER.
- **Authorization rule:** session scope; data-layer scoped by session Farmer ID (FR-LED-001/P-ISO-01; DBZ-01).
- **Query:** `page`, `size`, `from`, `to`.
- **Success:** `200` own ledger (date, invoice, product, quantity, rate, amount, payment, UTR) + outstanding.
- **Error:** 401/403/404.
- **Audit:** none (view).

### 9.8 GET /farmer/me/statements
- **Method/Endpoint:** `GET /api/v1/farmer/me/statements`
- **Purpose:** Own monthly statements by period (FR-PRT-009).
- **Authentication:** Bearer. **Required role:** FARMER.
- **Authorization rule:** session scope.
- **Query:** `page`, `size`.
- **Success:** `200` paged own statement summaries.
- **Error:** 401/403.
- **Audit:** none (view).

### 9.9 GET /farmer/me/statements/{statementId}/pdf
- **Method/Endpoint:** `GET /api/v1/farmer/me/statements/{statementId}/pdf`
- **Purpose:** Download own statement PDF (`[SOURCE §13]`; `docs/07` §15).
- **Authentication:** Bearer. **Required role:** FARMER.
- **Authorization rule:** statement's Farmer ID MUST equal session Farmer ID; else `404` (P-ISO-01/03). PDF served only after ownership check.
- **Path:** `statementId`.
- **Success:** `200` `application/pdf` artifact.
- **Error:** 401/403/404.
- **Audit:** denial logging (P-ISO-04).

### 9.10 GET /farmer/me/notifications
- **Method/Endpoint:** `GET /api/v1/farmer/me/notifications`
- **Purpose:** Own notification history (FR-PRT-010).
- **Authentication:** Bearer. **Required role:** FARMER.
- **Authorization rule:** session scope.
- **Query:** `page`, `size`.
- **Success:** `200` paged own notifications.
- **Error:** 401/403.
- **Audit:** none (view).

### 9.11 PATCH /farmer/me/notifications/{notificationId}/read
- **Method/Endpoint:** `PATCH /api/v1/farmer/me/notifications/{notificationId}/read`
- **Purpose:** Mark own notification read (`[PROPOSED]` P-FF-02).
- **Authentication:** Bearer. **Required role:** FARMER.
- **Authorization rule:** notification belongs to session farmer.
- **Success:** `204`.
- **Error:** 401/403/404.
- **Audit:** §17 (view-state change).

---

## 10. Group 6 — Products

### 10.1 GET /products
- **Method/Endpoint:** `GET /api/v1/products`
- **Purpose:** List selectable products/units for procurement (FR-PROC-005; `[SOURCE §5]`).
- **Authentication:** Bearer. **Required role:** EMPLOYEE or SUPER_ADMIN.
- **Authorization rule:** procurement-scope read.
- **Query:** `page`, `size`, `activeOnly`.
- **Success:** `200` paged products (name, unit, status; quality/grade future `[SOURCE §20]`).
- **Error:** 401/403.
- **Audit:** none (read).

### 10.2 POST /products
- **Method/Endpoint:** `POST /api/v1/products`
- **Purpose:** Create product (catalogue governance OQ-05).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN (matrix per OQ-05).
- **Body:** name, unit.
- **Validation:** unique name; unit valid.
- **Success:** `201`.
- **Error:** 400/409.
- **Audit:** §17 (mandatory) — master change.

### 10.3 PUT /products/{productId}
- **Method/Endpoint:** `PUT /api/v1/products/{productId}`
- **Purpose:** Update product/unit/status.
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Success:** `200`. **Error:** 400/404/409.
- **Audit:** §17 (mandatory).

---

## 11. Group 7 — Procurement

### 11.1 POST /procurement/preview (`[PROPOSED]` helper)
- **Method/Endpoint:** `POST /api/v1/procurement/preview`
- **Purpose:** Compute gross = quantity × rate, net = gross − deduction without persisting (supports guided entry `[SOURCE §5]`).
- **Authentication:** Bearer. **Required role:** EMPLOYEE/SUPER_ADMIN.
- **Authorization rule:** employee procurement scope; farmer must be selectable (Phase 2: in assigned area, E-AR).
- **Body:** productId, quantity, unit, rate, deduction?
- **Validation:** positive qty/rate; deduction ≤ gross (proposed guard).
- **Success:** `200` computed amounts.
- **Error:** 400/403.
- **Audit:** none (read-only compute).

### 11.2 POST /procurement
- **Method/Endpoint:** `POST /api/v1/procurement`
- **Purpose:** Confirm purchase and **generate the invoice automatically** (FR-PROC-001/004; workflow end `[SOURCE §5, §6]`). No invoice number is accepted/typed by the employee (PINV-BR-10).
- **Authentication:** Bearer. **Required role:** EMPLOYEE or SUPER_ADMIN.
- **Authorization rule:** employee can transact for selectable farmer (`[SOURCE §16]`); Phase 2 area-scoped (E-AR).
- **Body:** farmerId (server-validated against employee scope), productId, quantity, unit, rate, deduction?, deductionReason?, remarks?.
- **Validation:** farmer exists & in scope; product valid; qty>0, rate>0; deduction≤gross; amount computed server-side (needs `quantity × rate`, `[SOURCE §5]`); attribution = session employee (PRD-PROC-004).
- **Success:** `201` transaction confirmation per `[SOURCE §7]`: Farmer, Invoice, Date, Product, Quantity, Rate, Total, Employee; server-assigned invoice number.
- **Error:** 400 (validation), 403 (farmer out of employee scope), 409 (concurrent numbering retry), 404.
- **HTTP status codes:** 201, 400, 401, 403, 404, 409.
- **Audit:** §17 (mandatory) — creation + employee attribution (PRD-PROC-004); invoice generation.

### 11.3 GET /procurement/{procurementId}
- **Method/Endpoint:** `GET /api/v1/procurement/{procurementId}`
- **Purpose:** View a procurement transaction.
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN (employee own records per matrix; OQ-09).
- **Authorization rule:** admin full; employee own-created or area-scoped per matrix.
- **Path:** `procurementId`.
- **Success:** `200` transaction detail.
- **Error:** 401/403/404.
- **Audit:** none (read).

### 11.4 GET /procurement
- **Method/Endpoint:** `GET /api/v1/procurement`
- **Purpose:** List/filter procurement (register `[SOURCE §14]`).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN (matrix-dependent for employees).
- **Authorization rule:** admin full; employee Phase 2 area-scoped.
- **Query:** `from`, `to`, `farmerId`, `productId`, `employeeId`, `page`, `size`.
- **Success:** `200` paged list.
- **Error:** 400/401/403.
- **Audit:** none (read).

---

## 12. Group 8 — Invoices

### 12.1 GET /invoices
- **Method/Endpoint:** `GET /api/v1/invoices`
- **Purpose:** Invoice register (`[SOURCE §14]`).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN (per matrix).
- **Authorization rule:** admin full; employee sole access per matrix (OQ-09/RBAC-02).
- **Query:** `from`, `to`, `farmerId`, `status`, `page`, `size`.
- **Success:** `200` paged invoices.
- **Error:** 400/401/403.
- **Audit:** none (read).

### 12.2 GET /invoices/{invoiceNumber}
- **Method/Endpoint:** `GET /api/v1/invoices/{invoiceNumber}`
- **Purpose:** Single invoice detail (admin/employee).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN; EMPLOYEE only for own/assigned-area invoices (matrix).
- **Authorization rule:** admin full; employee scoped (own confirmations `[SOURCE §7]`; Phase 2 area).
- **Path:** `invoiceNumber` (`F-0001-17`).
- **Success:** `200` invoice detail.
- **Error:** 401/403/404.
- **Audit:** none (read).

### 12.3 POST /invoices/{invoiceNumber}/cancel
- **Method/Endpoint:** `POST /api/v1/invoices/{invoiceNumber}/cancel`
- **Purpose:** Cancel invoice — retained in system, number never reused (FR-INV-004, `[SOURCE §6]`).
- **Authentication:** Bearer. **Required role:** PERMISSION-defined (actor undefined — OQ-08; see RBAC-03).
- **Authorization rule:** as granted by matrix.
- **Path:** `invoiceNumber`. **Body:** `{ "reason": string }`.
- **Validation:** invoice exists & not already cancelled; state transition valid; compensation/ledger treatment undefined (OQ-08).
- **Success:** `200` cancelled invoice (status=cancelled, original preserved).
- **Error:** 400, 403, 404, 409 (already cancelled).
- **HTTP status codes:** 200, 400, 401, 403, 404, 409.
- **Audit:** §17 (mandatory) — cancellation incl. actor, reason, original/new (FR-INV-004/006).

### 12.4 GET /invoices/{invoiceNumber}/pdf
- **Method/Endpoint:** `GET /api/v1/invoices/{invoiceNumber}/pdf`
- **Purpose:** Printable/PDF invoice (OQ-07; `[PROPOSED]` until approved).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN; EMPLOYEE own; FARMER own via portal.
- **Authorization rule:** ownership/scope check; farmer must own invoice (P-ISO).
- **Success:** `200` PDF.
- **Error:** 401/403/404.
- **Audit:** denial logging only.

---

## 13. Group 9 — Payments

### 13.1 POST /payments
- **Method/Endpoint:** `POST /api/v1/payments`
- **Purpose:** Create a payment with invoice allocation(s) (FR-PAY-001; `[SOURCE §10]`).
- **Authentication:** Bearer. **Required role:** PERMISSION-defined — creator role undefined (Q-PAY-010; accounts staff implied `[SOURCE §11]`).
- **Authorization rule:** accounts/admin scope.
- **Body:** `{ farmerId, amount, date, mode, bankReference?, utr?, allocations: [ { invoiceNumber, amount } ], remarks? }`.
- **Validation:** amount>0; sum(allocations)=amount (guard `[PROPOSED]` Q-PAY-006); allocations to same farmer's invoices (DI-10); over-allocation ≤ due blocked (FR-PAY-001/005 guards).
- **Success:** `201` payment record + allocation rows.
- **Error:** 400 (sum mismatch/over-allocation), 403, 404, 409 (duplicate via Idempotency-Key).
- **HTTP status codes:** 201, 400, 401, 403, 404, 409.
- **Audit:** §17 (mandatory).

### 13.2 GET /payments/{paymentId}
- **Method/Endpoint:** `GET /api/v1/payments/{paymentId}`
- **Purpose:** Payment detail (with allocations, UTR, status).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN/accounts (Q-PAY-010).
- **Authorization rule:** accounts/admin scope.
- **Path:** `paymentId`.
- **Success:** `200` payment detail.
- **Error:** 401/403/404.
- **Audit:** none (read).

### 13.3 GET /payments
- **Method/Endpoint:** `GET /api/v1/payments`
- **Purpose:** Payment register/list (`[SOURCE §14]`).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN/accounts.
- **Query:** `from`, `to`, `farmerId`, `status`, `page`, `size`.
- **Success:** `200` paged payments.
- **Error:** 400/401/403.
- **Audit:** none (read).

### 13.4 PATCH /payments/{paymentId}/status
- **Method/Endpoint:** `PATCH /api/v1/payments/{paymentId}/status`
- **Purpose:** Manual status adjustment (manual reconciliation fallback — undefined; Q-PAY-005, `[PROPOSED]` until approved).
- **Authentication:** Bearer. **Required role:** accounts/admin.
- **Body:** `{ "status": matched|unmatched|failed|pending|duplicate }`.
- **Validation:** valid transition; matched only where allocation exists.
- **Success:** `200`. **Error:** 400/403/404/409.
- **Audit:** §17 (mandatory) — status change original/new.

---

## 14. Group 10 — Reconciliation

### 14.1 GET /reconciliation/queue
- **Method/Endpoint:** `GET /api/v1/reconciliation/queue`
- **Purpose:** Exception/Reconciliation Queue for accounts staff (FR-REC-004, `[SOURCE §11]`).
- **Authentication:** Bearer. **Required role:** accounts staff (permission-defined).
- **Authorization rule:** accounts scope.
- **Query:** `status` (unmatched/failed/pending/duplicate), `page`, `size`.
- **Success:** `200` paged queue items.
- **Error:** 401/403.
- **Audit:** none (read).

### 14.2 GET /reconciliation/queue/{paymentId}
- **Method/Endpoint:** `GET /api/v1/reconciliation/queue/{paymentId}`
- **Purpose:** Queue item detail (payment + UTR + attempted match context).
- **Authentication:** Bearer. **Required role:** accounts staff.
- **Success:** `200`. **Error:** 401/403/404.
- **Audit:** none (read).

### 14.3 POST /reconciliation/queue/{paymentId}/resolve
- **Method/Endpoint:** `POST /api/v1/reconciliation/queue/{paymentId}/resolve`
- **Purpose:** Resolve a queue item (e.g., match to allocation, re-initiate, adjust duplicate) (FR-REC-004/005).
- **Authentication:** Bearer. **Required role:** accounts staff.
- **Authorization rule:** accounts scope; actions must be valid for the item state (Q-PAY-003).
- **Body:** `{ "resolutionType": "match|retry|adjust|close", "allocations"?: [ {invoiceNumber, amount} ], "notes"?: }`.
- **Validation:** state-transition validity (§29–36 `docs/09`); matched resolution triggers ledger update + notification (`[SOURCE §11, §12]`).
- **Success:** `202` accepted / `200` resolved.
- **Error:** 400, 403, 404, 409 (invalid transition).
- **HTTP status codes:** 200/202, 400, 401, 403, 404, 409.
- **Audit:** §17 (mandatory) — queue actions (FR-REC-004).

### 14.4 POST /reconciliation/webhook (provider callback — placeholder)
- **Method/Endpoint:** `POST /api/v1/reconciliation/webhook`
- **Purpose:** Receive provider payment status/UTR where supported (`[SOURCE §11]`; FR-REC-002). Vendor-neutral payload placeholder.
- **Authentication:** Provider mechanical behavior — but credential/signing model undefined (Q-PAY-001); MUST be authenticated after vendor selection.
- **Required role:** SYSTEM (service credential / signed webhook).
- **Authorization rule:** verified provider origin only.
- **Body:** provider-agnostic: `{ "paymentRef", "status", "utr"? }` after mapping decision (Q-PAY-001).
- **Validation:** reference resolvable; status ∈ mandated set.
- **Success:** `200` ack (idempotent).
- **Error:** 400/401/409.
- **HTTP status codes:** 200, 400, 401, 409.
- **Audit:** §17 (mandatory) — integration activity (FR-REC-001/002).

### 14.5 GET /reconciliation/utr-register
- **Method/Endpoint:** `GET /api/v1/reconciliation/utr-register`
- **Purpose:** UTR register report data (`[SOURCE §14]`).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN/accounts.
- **Query:** `from`, `to`, `farmerId`.
- **Success:** `200` UTR register.
- **Error:** 400/401/403.
- **Audit:** none (read).

---

## 15. Group 11 — Ledger

### 15.1 GET /ledger/farmers/{farmerId}
- **Method/Endpoint:** `GET /api/v1/ledger/farmers/{farmerId}`
- **Purpose:** Farmer ledger (admin/employee view). **Admin-only equivalent of the farmer self-view.** Not available to the farmer portal (which uses §9.7).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN (employee per matrix OQ-09/RBAC-02).
- **Authorization rule:** admin full; employee Phase 2 restricted to assigned area (E-AR) — farmer must belong to employee's area.
- **Path:** `farmerId`.
- **Query:** `from`, `to`, `page`, `size`.
- **Success:** `200` ledger per `[SOURCE §9]`.
- **Error:** 401/403/404.
- **Audit:** none (read).

### 15.2 GET /ledger/farmers/{farmerId}/outstanding
- **Method/Endpoint:** `GET /api/v1/ledger/farmers/{farmerId}/outstanding`
- **Purpose:** Farmer's outstanding balance (derived; FR-LED-004).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN (per matrix).
- **Authorization rule:** as §15.1.
- **Success:** `200` `{ farmerId, outstanding }`.
- **Error:** 401/403/404.
- **Audit:** none (computed).

---

## 16. Group 12 — Statements

### 16.1 GET /statements
- **Method/Endpoint:** `GET /api/v1/statements`
- **Purpose:** Statement register incl. delivery status (`[SOURCE §13]`; FR-MST-005).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Query:** `periodFrom`, `periodTo`, `status`, `page`, `size`.
- **Success:** `200` paged statements (generated/sent/delivered/failed/retry).
- **Error:** 400/401/403.
- **Audit:** none (read).

### 16.2 GET /statements/{statementId}
- **Method/Endpoint:** `GET /api/v1/statements/{statementId}`
- **Purpose:** Statement detail (balances, status history).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Success:** `200`. **Error:** 401/403/404.
- **Audit:** none (read).

### 16.3 GET /statements/{statementId}/pdf
- **Method/Endpoint:** `GET /api/v1/statements/{statementId}/pdf`
- **Purpose:** Statement PDF (admin; reprint OQ-14). Farmer uses §9.9 URL.
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Success:** `200` PDF. **Error:** 401/403/404.
- **Audit:** §17 where required (privileged download).

### 16.4 POST /statements/generate (manual trigger — `[PROPOSED]`)
- **Method/Endpoint:** `POST /api/v1/statements/generate`
- **Purpose:** Re-run / manual statement generation for a period. Note: source mandates automatic generation on the 1st (`[SOURCE §13]`); manual re-run is proposed for ops/retry handling (OQ-14).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Body:** `{ "periodStart", "periodEnd" }`.
- **Validation:** closed period.
- **Success:** `202` job accepted.
- **Error:** 400/403.
- **Audit:** §17 (mandatory).

---

## 17. Group 13 — WhatsApp

### 17.1 GET /whatsapp/messages
- **Method/Endpoint:** `GET /api/v1/whatsapp/messages`
- **Purpose:** Message/delivery log (admin) — incl. statement statuses (`[SOURCE §13]`; FR-WH-003; scope for other types Q-WH-03).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Query:** `type`, `status`, `from`, `to`, `page`, `size`.
- **Success:** `200` paged messages (mobile masked).
- **Error:** 401/403.
- **Audit:** none (read).

### 17.2 GET /whatsapp/messages/{messageId}
- **Method/Endpoint:** `GET /api/v1/whatsapp/messages/{messageId}`
- **Purpose:** Message detail + delivery status history.
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Success:** `200`. **Error:** 401/403/404.
- **Audit:** none (read).

### 17.3 POST /whatsapp/retry (`[PROPOSED]`)
- **Method/Endpoint:** `POST /api/v1/whatsapp/retry`
- **Purpose:** Manual re-send/retry of failed messages (statement retry status exists `[SOURCE §13]`; manual re-send is a proposed enhancement — F-06).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Body:** `{ "messageIds": [...] }` or `{ "statementId": ... }`.
- **Validation:** items in failed/retry state.
- **Success:** `202`.
- **Error:** 400/403/404.
- **Audit:** §17 (mandatory).

### 17.4 POST /whatsapp/webhook (delivery callback — placeholder)
- **Method/Endpoint:** `POST /api/v1/whatsapp/webhook`
- **Purpose:** Receive delivery-status updates from the WhatsApp provider (statuses `[SOURCE §13]`).
- **Authentication:** Provider signing — model undefined (Q-WH-01); MUST be authenticated.
- **Required role:** SYSTEM.
- **Authorization rule:** verified provider.
- **Body:** provider-agnostic status payload (mapped after vendor selection).
- **Success:** `200` ack.
- **Error:** 400/401.
- **Audit:** §17 (mandatory) — delivery status transitions.

### 17.5 POST /whatsapp/webhook/inbound (Phase 2 — chatbot)
- **Method/Endpoint:** `POST /api/v1/whatsapp/webhook/inbound`
- **Purpose:** Receive farmer chatbot queries (Phase 2, `[SOURCE §21]`: My Outstanding, My Ledger, My Purchases, My Payments, Last Payment, Last Purchase, Invoice Details, Statement, Payment Status).
- **Authentication:** Provider signing; Farmer identity = registered mobile (`[SOURCE §21]`; OQ-13).
- **Required role:** SYSTEM.
- **Authorization rule:** resolve mobile → Farmer ID; responses scoped to that farmer (own data).
- **Body:** provider-agnostic inbound message payload (post vendor selection).
- **Success:** `200` ack (response sent asynchronously).
- **Error:** 400/401.
- **Audit:** §17 — query/response logging (Proposed; Q-WH-07).
- **Phase 2.**

---

## 18. Group 14 — Notifications

### 18.1 GET /notifications
- **Method/Endpoint:** `GET /api/v1/notifications`
- **Purpose:** Notification register (admin view of outbound notification history).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Query:** `type`, `from`, `to`, `page`, `size`.
- **Success:** `200` paged notifications.
- **Error:** 401/403.
- **Audit:** none (read).

### 18.2 GET /notifications/farmers/{farmerId}
- **Method/Endpoint:** `GET /api/v1/notifications/farmers/{farmerId}`
- **Purpose:** Farmer's notification history (admin). Farmer uses `GET /farmer/me/notifications` (§9.10).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Authorization rule:** full scope.
- **Success:** `200`. **Error:** 401/403/404.
- **Audit:** none (read).

---

## 19. Group 15 — Reports

### 19.1 GET /reports/{reportType}
- **Method/Endpoint:** `GET /api/v1/reports/{reportType}`
- **Purpose:** Standard reports (`[SOURCE §14]`): farmer-wise, payment register, UTR register, monthly summary, product-wise, employee-wise, (Phase 2 area-wise).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN (per matrix `[SOURCE §16]`).
- **Authorization rule:** admin full; employee per matrix; Phase 2 employee area-scoped (FRPT-02, E-AR).
- **Path:** `reportType` (enum of §13 report types).
- **Query:** `from`, `to`, `farmerId`, `productId`, `employeeId` (report-specific), pagination.
- **Success:** `200` report payload.
- **Error:** 400 (unknown type), 403, 404.
- **Audit:** §17 where privileged (report/export logging per `docs/13` §14; exact granularity Q-RPT/undefined).

### 19.2 GET /reports/{reportType}/export
- **Method/Endpoint:** `GET /api/v1/reports/{reportType}/export`
- **Purpose:** Export report (PDF/Excel/CSV where appropriate) (`[SOURCE §14]`; FR-RPT-004).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN (per matrix).
- **Authorization rule:** same data-scope as §19.1 (Phase 2 area filter applied server-side).
- **Query:** same filters + `format` (pdf|xlsx|csv).
- **Success:** `200` file download.
- **Error:** 400/403/404.
- **Audit:** §17 — export logged (docs/13 §14).

---

## 20. Group 16 — Audit

### 20.1 GET /audit-logs
- **Method/Endpoint:** `GET /api/v1/audit-logs`
- **Purpose:** Audit trail browser (admin) (`[SOURCE §2, §17]`).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Authorization rule:** full scope (employee read prohibited — RBAC-02/null).
- **Query:** `actorId`, `entity`, `action`, `from`, `to`, `page`, `size`.
- **Success:** `200` paged audit records (original/new, IP, device).
- **Error:** 400/401/403.
- **Audit:** read events logged if policy requires (not specified).

### 20.2 GET /audit-logs/{entity}/{recordRef}
- **Method/Endpoint:** `GET /api/v1/audit-logs/{entity}/{recordRef}`
- **Purpose:** Full change history of a record (who changed what and when; `[SOURCE §17]`).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Path:** `entity` (e.g., invoice), `recordRef` (e.g., F-0001-17).
- **Success:** `200` ordered history.
- **Error:** 400/403/404.
- **Audit:** none (read; privileged).

---

## 21. Group 17 — Settings

### 21.1 GET /settings
- **Method/Endpoint:** `GET /api/v1/settings`
- **Purpose:** Read system settings (non-secret values).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Success:** `200` settings.
- **Error:** 401/403.
- **Audit:** none (read).

### 21.2 PUT /settings
- **Method/Endpoint:** `PUT /api/v1/settings`
- **Purpose:** Update operational settings (`[SOURCE §2]`; M18).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Body:** setting key/value map.
- **Validation:** keys exist; secret values never logged (§3.4).
- **Success:** `200`. **Error:** 400/403.
- **Audit:** §17 (mandatory) — config changes.

### 21.3 PUT /settings/integrations/banking
- **Method/Endpoint:** `PUT /api/v1/settings/integrations/banking`
- **Purpose:** Configure banking/API integration (provider-neutral; OQ-03/Q-PAY-001).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Body:** integration config/credentials (secured).
- **Success:** `200`. **Error:** 400/403.
- **Audit:** §17 (mandatory) — credentials never in audit payload.

### 21.4 PUT /settings/integrations/whatsapp
- **Method/Endpoint:** `PUT /api/v1/settings/integrations/whatsapp`
- **Purpose:** Configure WhatsApp provider (provider-neutral; OQ-04/Q-WH-01).
- **Authentication:** Bearer. **Required role:** SUPER_ADMIN.
- **Success:** `200`. **Error:** 400/403.
- **Audit:** §17 (mandatory).

---

## 22. Endpoint Summary (47 endpoints)

| Group | Endpoints |
|---|---|
| 1. Authentication | login, refresh, logout, farmer/otp/request, farmer/otp/verify, me (6) |
| 2. Admin | dashboard/kpis, permission-matrix GET/PUT, areas (P2) (3+1) |
| 3. Employee | GET/POST /employees, PUT /employees/{id}, area (P2) (3+1) |
| 4. Farmer | search, POST /farmers, GET/PUT /farmers/{id}, KYC POST/GET, area (P2) (6+1) |
| 5. Farmer Portal | dashboard, profile, purchases, invoices, invoice detail, payments, ledger, statements, statement pdf, notifications, notification read (11) |
| 6. Products | GET/POST /products, PUT /products/{id} (3) |
| 7. Procurement | preview, POST /procurement, GET /procurement/{id}, GET /procurement (4) |
| 8. Invoices | GET /invoices, GET /invoices/{n}, POST cancel, GET pdf (4) |
| 9. Payments | POST /payments, GET /payments/{id}, GET /payments, PATCH status (4) |
| 10. Reconciliation | queue GET×2, resolve, webhook, utr-register (5) |
| 11. Ledger | ledger farmer, outstanding (2) |
| 12. Statements | GET ×2, pdf, generate (4) |
| 13. WhatsApp | messages ×2, retry, webhook, inbound-webhook P2 (4+1) |
| 14. Notifications | GET ×2 (2) |
| 15. Reports | report, export (2) |
| 16. Audit | audit-logs ×2 (2) |
| 17. Settings | GET/PUT settings, banking, whatsapp (4) |

---

## 23. Open Questions — API

| ID | Question | Origin |
|---|---|---|
| Q-API-01 | Farmer authentication method (OTP path endpoints are `[PROPOSED]`) | OQ-02 |
| Q-API-02 | Payment creator role for POST /payments | Q-PAY-010 |
| Q-API-03 | Wallet Azure ID/token model (access+refresh; expiry) | FR-AUTH-005 |
| Q-API-04 | Webhook credential/signing models for banking & WhatsApp | Q-PAY-001, Q-WH-01 |
| Q-API-05 | Manual reconciliation & status-adjust endpoints (approved vs proposed) | Q-PAY-005 |
| Q-API-06 | Invoice PDF and report-export audit granularity | OQ-07; `docs/13` §16 |
| Q-API-07 | Query pagination/search parameter set per list endpoint | Undefined |
| Q-API-08 | Timezone for date filters (ISO offsets) | Q-DB-09 / OQ-14 |
| Q-API-09 | Chatbot inbound API response contract (Phase 2) | Q-WH-05 |
| Q-API-10 | Employee access to specific report endpoints | OQ-09 / Q-RBAC-02 |

---

*End of REST API Specification v1.0. Next in sequence: `17_SECURITY_SPECIFICATION.md`.*