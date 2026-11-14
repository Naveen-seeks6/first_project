# TableBite — Technical Requirements and Architecture (TRD)

**Status:** Proposed architecture for implementation  
**Target:** Single-restaurant production system

## 1. Current repository baseline

The repository currently contains a vanilla HTML/CSS/JavaScript customer interface and admin interface, a Node.js HTTP server, a native SQLite database module, seed data, QR generation, and a Supabase REST adapter.

The existing code is a useful prototype, not yet safe for public production. Initial code inspection identified the following release blockers:

- Authentication accepts hard-coded demo credentials/PIN and issues a timestamp-shaped token that is not a verified session.
- Admin JavaScript includes an offline authentication fallback.
- Several sensitive API routes (orders listing/status, payment marking, settings, menu mutation, analytics) do not consistently enforce authentication/authorization.
- The payment confirmation endpoint marks an order paid based on a caller request.
- PayU callback verification contains a bypass when the configured salt equals the demo salt.
- Payment callback URLs are built with plain HTTP and the request Host header.
- Default merchant/payment and restaurant settings are embedded in source.
- QR link generation uses HTTP and should use a configured canonical HTTPS base URL.
- SQLite file persistence and WAL sidecar files need deployment/backup planning; live database artifacts should not be committed.
- Existing order creation has validation gaps (quantity bounds, payment method allowlist, duplicate add-ons, idempotency, and transaction boundaries).

These findings must be fixed before public launch. Rotate any real credentials that have been committed. Do not copy repository secrets into deployment configuration without rotating them.

## 2. Target architecture

### Frontend
- Retain the current vanilla JavaScript, semantic HTML, and CSS initially to avoid a risky full rewrite.
- Separate customer, owner/admin, kitchen, and cashier experiences by route/view and permission.
- Centralize API client, session handling, input validation, and error presentation.
- Never embed database service keys or payment secrets in frontend assets.

### Backend
- Node.js 22+ HTTP application initially, refactored into modules: routes/controllers, services, repositories, validation, authentication/authorization, payment adapter, and configuration.
- Prefer a maintained HTTP framework (Fastify or Express) after dependency/security review; keep API contracts stable during migration.
- All business rules and authoritative calculations run server-side.
- Central error handling, request IDs, structured logs, health/readiness endpoints, and strict request body limits.
- Configure allowed origins explicitly; do not use wildcard CORS for authenticated APIs.

### Database
- Recommended production target: managed PostgreSQL (Supabase PostgreSQL is a candidate) for durable storage, concurrency, backups, and future operational growth.
- Keep SQLite only for local development/test or a deliberate single-instance deployment with persistent disk and tested backup strategy.
- Use versioned SQL migrations; do not rely on ad-hoc schema creation at application startup in production.
- Use transactions for order plus line-item creation and payment reconciliation.
- Use integer minor currency units (paise) or a fixed-precision decimal strategy; avoid binary floating-point money arithmetic.
- Add staff, sessions, audit events, payment transactions, order status history, and idempotency records.
- Add foreign keys, unique constraints, indexes, and retention policies.

### Realtime
- Phase 1: authenticated polling with backoff and efficient updated-since queries.
- Phase 2: PostgreSQL/Supabase Realtime or authenticated WebSocket/SSE when needed.
- Realtime is a notification mechanism; clients must reload authoritative server state.

### Payments
- Isolate payment-provider code behind an adapter.
- Select one provider after merchant account and webhook capability are confirmed.
- Store payment intent/transaction IDs and immutable provider event references.
- Verify callback signature/hash and reconcile amount, currency, merchant, order, and transaction state.
- Idempotently process webhook events; do not trust browser redirects or client-supplied transaction references.
- Cash collection is a separate cashier-authorized ledger event.
- Never log secrets, full payment payloads containing personal data, or authentication tokens.

## 3. Proposed data model

- `staff_users`: id, email, display_name, password_hash, role, active, created_at, updated_at.
- `staff_sessions`: id, staff_user_id, token_hash, expires_at, revoked_at, created_at.
- `categories`: id, name, icon, display_order, active.
- `products`: id, category_id, name, description, price_minor, image_url, dietary flags, availability, created_at, updated_at.
- `product_portions`: id, product_id, name, price_minor, active.
- `product_addons`: id, product_id, name, price_minor, active.
- `restaurant_tables`: id, table_number, qr_token_hash, capacity, status, created_at.
- `orders`: id, order_number, table_id, customer_name, customer_phone, status, payment_status, payment_method, subtotal_minor, tax_minor, total_minor, idempotency_key, created_at, updated_at.
- `order_items`: id, order_id, product_id nullable, item_name_snapshot, unit_price_minor, quantity, selected_portion_snapshot, selected_addons_snapshot, item_total_minor, special_note.
- `order_status_events`: id, order_id, from_status, to_status, actor_staff_id nullable, created_at.
- `payment_transactions`: id, order_id, provider, provider_transaction_id, amount_minor, currency, status, verified_at, provider_event_id, created_at.
- `cash_collections`: id, order_id, amount_minor, cashier_staff_id, collected_at, note.
- `restaurant_settings`: key, value, updated_by, updated_at.
- `audit_logs`: id, actor_staff_id, action, resource_type, resource_id, metadata_redacted, created_at.
- `idempotency_records`: key, request_hash, response_reference, created_at.

The migration must preserve existing menu, order, and table data where valid. Define a tested export/import migration from the current SQLite database before switching production storage.

## 4. API boundaries

Public/guest:
- `GET /api/menu`
- `GET /api/tables/:tableNumber/resolve?token=...` (or equivalent QR token exchange)
- `POST /api/orders` (validated, idempotent)
- `GET /api/orders/:publicTrackingToken` (minimal customer-safe response)
- Payment intent creation endpoint for the selected provider.

Staff-authenticated:
- `POST /api/auth/login`, `POST /api/auth/logout`, `GET /api/auth/me`
- `GET /api/admin/orders`
- `PATCH /api/admin/orders/:id/status`
- `POST /api/admin/orders/:id/cash-collection`
- `GET/POST/PUT/DELETE /api/admin/menu...`
- `GET/POST/PATCH /api/admin/tables...`
- `GET/POST/PATCH /api/admin/staff...`
- `GET /api/admin/analytics`
- `GET/PUT /api/admin/settings`

Provider-only:
- `POST /api/payments/:provider/webhook` with provider signature verification and replay/idempotency protection.

Every route must declare its authentication and role policy. Never expose unrestricted administrative routes under public endpoints.

## 5. Security requirements

- Password hashing with Argon2id or bcrypt using reviewed parameters.
- Random opaque session IDs stored hashed server-side; secure, HttpOnly, SameSite cookies; expiry, rotation, and revocation.
- CSRF defenses for cookie-authenticated state-changing routes.
- Role authorization on the server, not merely hidden UI controls.
- Rate limiting for login, public order creation, and tracking endpoints.
- Strict schema validation and allowlists; parameterized SQL.
- HTTPS and secure headers; restrictive Content Security Policy after inline-script review.
- Config validation at startup; fail closed if production secrets are missing.
- Secrets stored only in deployment secret manager/environment; rotate exposed values.
- PII minimization, redacted logs, retention policy, and access audit trail.
- QR tokens are cryptographically random, stored hashed, and revocable.
- Do not expose sequential database IDs or customer phone numbers in public tracking responses.

## 6. Deployment and operations

- Managed Node hosting with HTTPS and environment secret management.
- Managed PostgreSQL with automated backups and restore testing.
- Separate development, staging, and production environments.
- CI checks: lint/static checks, unit tests, integration tests, dependency audit, migration validation.
- Health/readiness endpoint and uptime/error monitoring.
- Centralized structured logs with request correlation IDs.
- Documented deployment and rollback procedure.
- Domain and canonical base URL configured explicitly.
- Database migrations run as a controlled release step, not by every application replica.

## 7. Environment configuration (names only)

Examples:
- `NODE_ENV`
- `PORT`
- `APP_BASE_URL`
- `DATABASE_URL`
- `SESSION_SECRET`
- `PAYMENT_PROVIDER`
- Provider-specific merchant key/secret and webhook verification secret.

Do not commit actual values. Provide `.env.example` with placeholders only and ensure `.env`, database files, WAL/SHM files, and logs are ignored by Git.

## 8. Quality gates

- Unit tests for price/tax calculation, validation, role permissions, state transitions.
- Integration tests for order creation, transaction rollback, idempotency, staff sessions, payment callbacks, and cash collection.
- End-to-end tests for guest QR ordering and kitchen-to-customer status flow.
- Security tests proving anonymous users cannot access staff APIs and lower roles cannot escalate.
- Payment sandbox tests before live provider credentials are enabled.
- Backup restore test and deployment rollback rehearsal before go-live.
