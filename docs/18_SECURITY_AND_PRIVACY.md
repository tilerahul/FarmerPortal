# Agri Procurement & Farmer Management System
## Security & Privacy Specification

---

## 1. Document Information

| Item | Value |
|---|---|
| Product name | Agri Procurement & Farmer Management System |
| Document | Security & Privacy Specification |
| Version | v1.0 |
| Status | Draft — primary source = original requirement PDF (§2, §4, §16–§18); augmented by approved specs |
| Date | 2026-09-16 |
| Author role | Senior Application Security Architect |
| Purpose | Document authentication, authorization, data protection, application/API security, operations security and compliance/privacy posture across all three portals, with explicit separation of PDF-stated requirements, proposed practices, and open compliance questions |
| Primary source | `AgriProcurement & Farmer Management.pdf` (§18 Security; §17 Audit; §2, §4, §16 supporting) |

### Tag & status conventions

| Tag | Meaning |
|---|---|
| `[SOURCE §n]` | Requirement explicitly stated in the original PDF |
| `[NEW]` | Newly approved requirement not in the PDF |
| `[PROPOSED]` | Proposed security practice — needs approval before implementation; not a requirement |
| `[UNDEFINED]` / Open question | Not specified anywhere; tracked in §11 |
| D-E (deny) / F-R (farmer rights) | Behave identity tags used in later specs |

> **Compliance notice:** This document does NOT claim legal compliance (e.g., any privacy, banking or data-protection law). Compliance claims must be verified with legal counsel and re-verified against jurisdiction/law at implementation time (see §11 Q-SEC-20).

---

## 2. Standing Requirements (from the PDF) — summary table

| # | Topic | Status | Source |
|---|---|---|---|
| 1 | Authentication (login, denial of access, privileged accounts, strong passwords, no default passwords) | PDF requirement | `[SOURCE §2, §4, §18]` |
| 2 | Authorization: role-based access, deny-by-default permissions, least privilege | PDF requirement | `[SOURCE §4, §16]` |
| 3 | Data isolation: farmer sees only own records (server-enforced) | PDF + `[NEW]` + derived | `[SOURCE §22; docs/17 P-ISO]` |
| 4 | Bank information protection; sensitive data masking | PDF requirement | `[SOURCE §18]`; display policy OQ-18 |
| 5 | Encryption in transit / at rest | PDF requirement (storage encryption stated) | `[SOURCE §2, §18]`; transport details `[PROPOSED]` |
| 6 | Audit logging (who/what/when/change) | PDF requirement | `[SOURCE §2, §17]` |
| 7 | Farmer authentication / OTP security | `[NEW]`; mechanism `[UNDEFINED]` | FR-PRT-001, OQ-02 |
| 8 | Session & token management, rate limiting, file uploads, backups, monitoring, incident handling, retention | Mostly `[PROPOSED]` | see sections |

---

## 3. Authentication — Summary

| Aspect | Requirement status | Detail |
|---|---|---|
| Staff (employee/admin) login | `[SOURCE §2]` "Login for the user", access denied to unauthenticated users | Requires login; no anonymous access |
| Privileged accounts | `[SOURCE §2]` | Restricted/special handling; see §5 |
| Password policy | `[SOURCE §18]` | Strong password requirement stated; exact policy parameters `[UNDEFINED]` (Q-SEC-01) |
| No default passwords | `[SOURCE §4]` | Stated; enforced at account creation |
| Login attempts / throttling | `[PROPOSED]` | Account lockout, exponential backoff (FR-AUTH-006) |
| 2FA for privileged roles | `[PROPOSED]` (FR-AUTH-003) | Required proposal for SUPER_ADMIN |
| Farmer authentication | `[NEW]` flow; method `[UNDEFINED]` (OQ-02) | OTP-over-registered-mobile is the leading `[PROPOSED]` mechanism; see §4 |

---

## 4. Farmer Authentication

- **Requirement:** Farmer Portal requires login before any farmer data is shown (`[NEW]` FR-PRT-001). Session created only after successful authentication.
- **Mechanism `[UNDEFINED]`:** The PDF does not define farmer login. Candidate `[PROPOSED]`:
  1. OTP to the registered mobile (`[SOURCE §3]` mobile is the seed identity) — single factor with possession of the SIM as the binding factor.
  2. Username/password created by admin (`[SOURCE §18]` password guidance) — requires password reset/self-service to be designed.
- **Design principles `[PROPOSED]`:**
  - Login does not reveal whether a mobile number is registered (uniform responses; no enumeration).
  - OTP short-lived, single-use, numeric, with per-number and per-IP send limits (see §16).
  - No financial data rendered before the authenticate step completes.
  - Session derived from the authenticated Farmer ID; all downstream farmer APIs use `/farmer/me/…` (P-ISO-01); never accept a farmerID from the client for farmer data (`docs/16`).

---

## 5. OTP Security

> OTP security applies only if the OTP mechanism is approved (Q-SEC-02). Marked `[PROPOSED]`.

- **OTP properties:** cryptographically random, numeric, length ≥6, expiry (e.g., 5 minutes), single-use, cannot be replayed.
- **Transport:** delivered to registered mobile (SMS/WhatsApp — channel provider pending, provider-neutral; see `docs/11`).
- **Rate limits:** per-number send cap (e.g., ≤5/day), resend delay (e.g., 60s), per-number verify attempt cap (e.g., 5), per-IP cap; on breach → temp block + audit.
- **Anti-abuse:** uniform error timing; no OTP enumeration; OTP stored hashed; OTP never logged or returned in API responses.
- **Provisioning:`[PROPOSED]`** OTP resync/fallback when mobile is lost — pending Q-SEC-03.

---

## 6. Employee Authentication

- **Requirement (PDF):** Individual employee login (`[SOURCE §2]`), denial of access to unauthenticated requests (`[SOURCE §2]`), no default passwords (`[SOURCE §4]`).
- **Required behavior:** each employee has a unique logical user; sessions identify the employee for attribution (`[SOURCE §2]` "who performed it") and permission evaluation (`[SOURCE §16]`).
- **Password policy `[PROPOSED]`:** length + complexity per `[SOURCE §18]` "strong password"; expiry/rotation, history, and first-login forced change are pending Q-SEC-01/Q-SEC-04.
- **Account lifecycle `[PROPOSED]`:** disable on termination; immediate session revocation; password reset via admin for employee roles (Q-SEC-05).

---

## 7. Admin Authentication

- **Requirement (PDF):** Privileged access explicitly called out (`[SOURCE §2]`, `[SOURCE §16]` privileged users).
- **`[PROPOSED]` protections:**
  - SUPER_ADMIN and any role with permission-matrix write or user-management: mandatory 2FA (FR-AUTH-003) before privileged operations.
  - Separate privileged session flag; elevated actions (matrix edit, audit export, integration config) prompt re-authentication (step-up).
  - No default admin credentials (extends `[SOURCE §4]`); admin created via controlled onboarding (`[SOURCE §2]`).
  - Prudent session time-out and one-active-privileged-session rule (Q-SEC-06).

---

## 8. Authorization

- **Requirement (PDF):** Access limited by logged-in role (`[SOURCE §16]`); permissions deny by default (`[SOURCE §16]`); least privilege noted for employee access (`[SOURCE §4]`).
- **Model:** see `docs/12_RBAC_AND_AUTHORIZATION.md` — role × module × action × scope; matrix configurable at runtime (`[SOURCE §16]`).
- **Enforcement points `[PROPOSED]`:** authorization enforced **server-side** at the API layer for every request (BZ-01); the UI hides/disables actions purely as UX, never as security.
- **Deny-by-default:** a permission not granted = denied; unknown module/action → deny.
- **Matrix change auditing:** changes to the permission matrix are themselves audited (§20) with old/new captured.

---

## 9. Farmer Data Isolation

- **Requirement (PDF):** Farmers access only their own data (`[SOURCE §22]` in the area restriction context); reinforced as a prime farmer-portal rule (FR-PRT section; P-ISO-01…06).
- **Enforcement `[PROPOSED]` design:** data scoping is resolved **in the data layer** (query predicates derived from session identity), never trusted from client-supplied IDs; documented as DBZ-01 (own-data) and TC-05 (`docs/15`).
- **Phase 2:** employees scoped to their assigned area at **database-level authorization** (`[SOURCE §22]`; DBZ-02/03) — block before any data returns, not filter-after-fetch (E-AR).
- **Failure posture `[PROPOSED]`:** a scoped query that returns affected rows ≠ expected rows must fail closed (no partial leakage); cross-scope requests to existing records return 404 (no existence disclosure — P-ISO-02) unless the matrix explicitly allows.
- **Prevention of ID enumeration:** farmer-facing APIs are identity-first (`/farmer/me/...`), removing the enumeration surface entirely (`docs/16` §4, P-ISO-02).

---

## 10. Sensitive Farmer Data

Data classes and their handling:

| Data class | Requirement status | Handling |
|---|---|---|
| Farmer ID, name, address, mobile | `[SOURCE §3]` master data | Needed for operations; visible per role matrix |
| Bank account / IFSC / encoded bank details | `[SOURCE §18]` protected; masking `[UNDEFINED]` levels | Masked by default UI policy (`OQ-18`); reveal-on-demand for staff with explicit permission; never shown to other farmers |
| KYC documents | `[UNDEFINED]` (future-ready, `[SOURCE §20]`) | Upload restricted to authorized roles; access log required (§11/§20) (Q-SEC-07) |
| Financial ledger, statements | `[SOURCE §9, §13]` | Own-data isolation (§9); audit of access `[PROPOSED]` for sensitive reads |
| UTR / bank refs | `[SOURCE §10, §11]` | Payee sees own; staff per matrix |

- **Principle `[PROPOSED]`:** least-display — render only fields the current role actually needs (supports `[SOURCE §4]` least privilege, `[SOURCE §16]` matrix, OQ-18).
- **Masking state `[UNDEFINED]`:** full vs partial mask, reveal policy, and whether staff may export unmasked are open (OQ-18).

---

## 11. Bank Information Protection

- **Requirement (PDF):** banking details protected; masked by default in the portal (`[SOURCE §18]` as evidenced by the masking-default in earlier specs); bank data may be modified only with appropriate authority (`[SOURCE §3]`).
- **`[PROPOSED]` controls:**
  - Display: masked by default; reveal requires role permission and returns masked/partial by default.
  - Storage: no plaintext keys; bank fields encrypted at rest (§13); secrets (integration/API) never in the UI.
  - Modification: change requires authorization + a fresh identity check (step-up) for admin, recorded in audit with before/after (masked diff).
  - Third-party bank integrations: credentials stored via secrets manager; outbound calls signed/authenticated (provider-neutral placeholders in `docs/16` Group 11); scope of stored bank-mapping data pending Q-PAY-001.
  - No bank data in logs, URLs, or error messages `[PROPOSED]`.
- **Open:** retention of bank data, third-party sub-processors, DPA obligations → Q-SEC-20.

---

## 12. KYC Documents

- **Requirement status:** KYC fields are future-ready (`[SOURCE §20]`); Farmer KYC entity exists in the data model (E-03/E-06 `docs/15`); upload/validation workflow `[UNDEFINED]` (OQ-10).
- **`[PROPOSED]` protections (if KYC is enabled):**
  - Upload via staff/admin roles only (never a generic public endpoint).
  - File-type allow-list + content-signature validation; size caps; virus scan queue (see §18).
  - Documents served/stored via a dedicated private object store; **signed, expiring URLs**; never browsable paths.
  - Every access, download, or delete audited with actor identity (traditional §20).
  - Masked rendering in farmer profile (`[SOURCE §16]` show-own-scope) with display policy per OQ-18.
- **Open:** retention period for KYC records → §26, Q-SEC-08.

---

## 13. Encryption in Transit

- **Requirement (PDF):** data encrypted in storage is stated (`[SOURCE §18]`); explicit in-transit clause is not present in the captured source — transport hardening is `[PROPOSED]` (Q-SEC-09).
- **`[PROPOSED]`:** TLS ≥1.2 (prefer 1.3) on all external and internal HTTP; HSTS; rejecting weak cipher suites; certificate management automation; no mixed content; farmer portal over HTTPS only (`[NEW]` FR-PRT base requirement).
- **Open:** whether any appliance/partner terminates TLS affecting WhatsApp/bank integrations → Q-SEC-10, Q-PAY-003.

---

## 14. Encryption at Rest

- **Requirement (PDF):** "System should support encrypted data storage to prevent data theft" (`[SOURCE §18]`). This is a **stated requirement**.
- **`[PROPOSED]` implementation points:**
  - Database: full-database or tablespace-level encryption (bank, KYC, credentials); keys via KMS, no keys on the app/DB server — provider/tech neutral (`docs/15`).
  - File/object store: server-side encryption for invoice PDFs and KYC documents.
  - Backups: encrypted (see §21) — since backups are copies of sensitive data.
  - Secrets (integration credentials) at rest only in the secrets manager (§23), never in source/config-in-repo.
- **Open:** is masking also required at backup-restore level (i.e., sanitised backup for non-prod)? → Q-SEC-11.

---

## 15. Session Management

- **Requirement (PDF):** login/enforced default deny (`[SOURCE §2]`); role-derived access (`[SOURCE §16]`). Session mechanics are `[PROPOSED]`.
- **`[PROPOSED]` rules:**
  - Server-side sessions or signed tokens; short-lived access tokens with refresh rotation.
  - Absolute expiry for admin/employee sessions (e.g., 8h idle / daily absolute) — pending Q-SEC-06.
  - Farmer sessions: lower privilege, longer comfortable window but shorter absolute timeout and no persistence of full financial data beyond page data; re-auth for sensitive reads where required (Q-SEC-02).
  - Session invalidation on: password change, role/permission change (permission changes revoke active sessions for affected role), account disable.
  - Single active privileged session `[PROPOSED]`; concurrent farmer sessions permitted but limited (Q-SEC-12).
  - Session fixation protection: rotate session id on privilege change. Logout destroys server-side session and client artifacts.

---

## 16. Token Security

- **`[PROPOSED]` (no PDF statement):**
  - Access tokens: short TTL (e.g., 15–30 min); refresh tokens: longer TTL, rotation on use, revocable server-side.
  - Tokens stored where appropriate (httpOnly secure cookie / memory) — never in localStorage for farmer portal `[PROPOSED]`.
  - Signing keys rotated; key per environment; no embedded keys in builds.
  - Token payload minimal (subject, roles/scope refs, exp); roles re-checked from server on each request (never trusted from token claims alone).
  - Refresh-token reuse detection → revoke whole session family + audit.
  - No tokens in URLs or logs.

---

## 17. Rate Limiting

- **`[PROPOSED]` (FR-AUTH-006 throttle + anti-abuse):**
  - Login/OTP endpoints: strictest limits (per identity, per IP, per device).
  - Farmer portal reads: identity-scoped limits; anomaly alerting.
  - Admin/employee: lower limits; imports/exports exempted by explicit burst policy.
  - Reconciliation/webhook inbound endpoints: integrity-first limits with signature verification.
  - Rate-limit responses are standardised; limits observable via headers; violations logged (§20).

---

## 18. API Security

- **Requirement (PDF):** access denied to unauthenticated (`[SOURCE §2]`), authorization per `[SOURCE §16]`, audit of actions (`[SOURCE §17]`).
- **`[PROPOSED]` layer:** see `docs/16_API_SPECIFICATION.md` —
  - All endpoints under `/api/v1`; unauthenticated returns 401; unauthorized 403; out-of-scope 404.
  - Versioned API; strict input validation (types, lengths, enums, ranges) at the boundary; reject unknown fields.
  - No sensitive parameters in query strings; PII in bodies only over TLS.
  - Idempotency keys for mutation endpoints (dedup OQ-13).
  - Structured error model: trace ID in responses, no stack traces / internal details to clients.
  - Inbound provider webhooks: signature verification + replay protection (pending Q-PAY-003 / Q-WH).
  - CORS restricted per portal origin; no wildcard with credentials.

---

## 19. File Upload Security

- **`[PROPOSED]` (needed for KYC §12 and any imports/exports):**
  - Allow-list of types + magic-byte content validation (never trust extension).
  - Per-file and per-request size limits; compressed/oversized rejection.
  - Store outside web root in private object storage; serve via signed URLs with TTL.
  - Filename sanitisation; no path traversal; unique stored filenames.
  - AV/file-scan queue before storage or before any downstream use (Q-SEC-13).
  - Uploads restricted to authorized roles (§12); every upload/download audited.

---

## 20. Audit Logging

- **Requirement (PDF):** audit trail that records user, action, date/time, and changed values (`[SOURCE §17]`; also `[SOURCE §2]` who/what/when). **Stated requirement.**
- **`[PROPOSED]` strengthening (extends `docs/14`):**
  - Append-only storage; staff (incl. admins) cannot edit/delete logs in the application path.
  - Log entries: actor (employee/admin/farmer), session, action, module, entity + record reference, old value, new value, IP/device, trace ID, timestamp (UTC).
  - Sensitive fields logged as masked/hashed values only.
  - Authentication events (login success/failure, OTP requests, lockouts), permission changes, configuration changes, data export/download are all auditable event classes.
  - Logs retained per §26; available to SUPER_ADMIN only (`docs/14`).
  - Clock synchronisation and log-write failure behavior defined (fail-safe vs fail-open pending Q-SEC-14).

---

## 21. Database Backups

- **`[PROPOSED]` (not explicitly in the captured PDF text — extend to Q-SEC-15; PDF only states encrypted storage §18):**
  - Scheduled full + incremental backups with point-in-time recovery window.
  - Backups encrypted at rest (aligning with `[SOURCE §18]` storage-encryption intent).
  - Off-site/secondary-region copy of the most recent full backup.
  - Backup metadata (window, integrity hash) validated regularly.

---

## 22. Backup Restoration Testing

- **`[PROPOSED]` (PDF has no statement):**
  - Scheduled restore drills (e.g., quarterly) on non-production targets; restore from real backup artefacts, not simulated.
  - Verification: data integrity checks, selective sampling, and an attested restore report.
  - RTO/RPO targets to be defined by the business (Q-SEC-15/Q-SEC-16).

---

## 23. Secrets Management

- **`[PROPOSED]` (needed to satisfy `[SOURCE §18]` storage-encryption intent credibly):**
  - Centralised secrets manager for: DB credentials, banking API keys, WhatsApp/bank provider credentials, OTP signing/verification keys, TLS private keys, refresh-token signing keys.
  - No secrets in code, config files in repo, or environment dumps; injection at runtime.
  - Rotation schedule + emergency rotation on suspected leak; rotation must not invalidate in-flight operations.
  - Role-segregated access to the secrets manager (only the application/service principal + designated admins).
  - Secrets auto-masked in logs and error surfaces (extends §14, §18).

---

## 24. Monitoring

- **`[PROPOSED]` (PDF silent; complements `[SOURCE §17]` audit requirement):**
  - Centralised application + infrastructure logs; structured; correlated by trace ID.
  - Security monitoring dashboards: auth failures, rate-limit hits, permission-denial surges, out-of-scope attempts, webhook/API anomalies, backup/encryption status.
  - Alerting on: brute-force/lockout waves, attempted enumeration against farmer endpoints, unexpected privileged actions, failed encryption/backup jobs, certificate expiry, secrets-rotation failures.
  - Log retention vs monitoring window sized per §26.
  - No PII in monitoring alerts where avoidable.

---

## 25. Incident Handling

- **`[PROPOSED]` (PDF silent — **not** a compliance claim):**
  - Defined incident tiers (security vs operational vs data) with response owners — pending Q-SEC-17.
  - Suspected data exposure → immediate containment: revoke sessions, rotate secrets, enable heightened logging, preserve audit artefacts.
  - Forensic copies taken from backup/encrypted artefacts before remediation; chain of custody for affected records.
  - Notification plan for affected farmers/employees and any required authorities implemented only through counsel (Q-SEC-20).
  - Post-incident review updates controls; changes logged in the audit trail.

---

## 26. Data Retention Considerations

- **Requirement status:** retention windows `[UNDEFINED]` (OQ-11). The PDF does not specify retention or deletion timelines.
- **`[PROPOSED]` policy scaffold (values pending OQ-11/Q-SEC-18):**
  - Master/ledger/statement data: business record — retained while legally required; define minimum retention based on financial-record norms (verify with counsel).
  - Audit logs: retention ≥ any legal minimum; immutable until expiry.
  - Farmer mobile / bank / KYC: retention tied to relationship + law; anonymise or purge after closure timelines TBD.
  - Logs/monitoring: segmented retention (raw vs derived).
  - Deletion orchestration: confirmed two-step (soft → purge) with audit of purge; no purge without authorisation.

---

## 27. Open Compliance & Privacy Questions

| ID | Question | State |
|---|---|---|
| Q-SEC-01 | Exact password parameters (length/complexity/expiry) | `[UNDEFINED]` |
| Q-SEC-02 | Farmer auth mechanism: OTP vs credentials (drives OTP section) | OQ-02 |
| Q-SEC-03 | OTP fallback when farmer loses/OTP mobile | `[UNDEFINED]` |
| Q-SEC-04 | Employee password self-service/reset flow | `[UNDEFINED]` |
| Q-SEC-05 | Admin password reset authority & break-glass path | `[UNDEFINED]` |
| Q-SEC-06 | Session timeout policy per role | `[UNDEFINED]` |
| Q-SEC-07 | Whether KYC access must be logged for every view | `[UNDEFINED]` (privacy posture) |
| Q-SEC-08 | KYC retention period | `[UNDEFINED]` |
| Q-SEC-09 | Is mandatory TLS formalised by client/appliance policy | `[UNDEFINED]` |
| Q-SEC-10 | Partner TLS termination for bank/WhatsApp integrations | `[UNDEFINED]` |
| Q-SEC-11 | Non-production sanitised backups required | `[UNDEFINED]` |
| Q-SEC-12 | Concurrent farmer sessions allowed | `[UNDEFINED]` |
| Q-SEC-13 | AV scanning vendor/policy for uploads | `[UNDEFINED]` |
| Q-SEC-14 | Audit-log write failure: fail-open vs fail-closed | `[UNDEFINED]` |
| Q-SEC-15 | Backup RPO / frequency | `[UNDEFINED]` |
| Q-SEC-16 | Backup RTO / restore target | `[UNDEFINED]` |
| Q-SEC-17 | Incident response ownership & escalation | `[UNDEFINED]` |
| Q-SEC-18 | Data retention windows incl. audit logs | OQ-11 |
| Q-SEC-19 | Which regulators/standards the client intends to comply with | `[UNDEFINED]` |
| Q-SEC-20 | Legal/compliance verification (privacy, banking, telecom/channel) | Must be verified by counsel — none claimed here |

---

## 28. Source Attribution Notes

- **Explicitly in the PDF:** stored-data encryption (`[SOURCE §18]`); strong passwords & no defaults (`[SOURCE §18, §4]`); login and anonymous-access denial (`[SOURCE §2]`); role-based/least-privilege access, deny-by-default (`[SOURCE §16, §4]`); privileged-user attention (`[SOURCE §2, §16]`); audit trail with actor/action/time/values (`[SOURCE §17]`); access scope governed by permissions (`[SOURCE §16]`); area-managed data domains (Phase 2) (`[SOURCE §22]`). Farmer own-data isolation derives from `[SOURCE §22]` + `[NEW]` farmer portal rules.
- **Everything else** (OTP mechanics, TLS specifics, sessions, tokens, rate limits, upload AV, backups, restore testing, secrets, monitoring, incident handling, retention) is **`[PROPOSED]`** or **`[UNDEFINED]`** and must be approved before implementation.

---

*End of Security & Privacy Specification v1.0. Next in sequence: `19_NON_FUNCTIONAL_REQUIREMENTS_AND_SCALABILITY.md` (per source §19–§20).*