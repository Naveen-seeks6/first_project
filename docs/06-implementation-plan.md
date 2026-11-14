# TableBite — Production Implementation Plan

## Delivery approach

Build in small, reviewable milestones on feature branches. Preserve the working customer UI where practical. Do not deploy publicly until the security and payment release gates pass.

## Phase 0 — Foundation and repository hygiene
- [x] Confirm single-restaurant business model.
- [x] Confirm UPI and cash-at-counter payment scope.
- [x] Confirm owner/admin/kitchen/cashier role model.
- [x] Document product requirements and technical architecture.
- [ ] Inspect all routes and client API calls; produce endpoint inventory.
- [ ] Add .gitignore for secrets, environment files, database/WAL files, logs, and local artifacts.
- [ ] Rotate any real credentials exposed in Git history/repository.
- [ ] Add .env.example with placeholders and startup configuration validation.
- [ ] Establish test/lint/CI baseline.

## Phase 1 — Secure application foundation
- [ ] Refactor server into route, service, repository, validation, auth, and config modules.
- [ ] Implement staff users, secure password hashes, sessions, logout, expiry, and revocation.
- [ ] Add role-based authorization to every protected endpoint.
- [ ] Remove demo PIN/password credentials and offline/local-token fallback.
- [ ] Protect settings, analytics, menu mutation, order listing, status mutation, and payment operations.
- [ ] Add request validation, consistent errors, rate limits, secure headers, and audit logging.
- [ ] Add tests proving anonymous and unauthorized access is rejected.

## Phase 2 — Data layer and migration
- [ ] Choose managed PostgreSQL provider and hosting region.
- [ ] Design and version schema migrations.
- [ ] Migrate existing categories, products, tables, orders, and order items from SQLite.
- [ ] Add staff, sessions, order events, payment ledger, cash collection, and idempotency records.
- [ ] Convert money storage/calculation to integer paise or fixed precision.
- [ ] Add transaction and backup/restore tests.

## Phase 3 — Customer ordering reliability
- [ ] Validate table QR token and support revocation/rotation.
- [ ] Validate item IDs, availability, portions, add-ons, quantity limits, and customer inputs.
- [ ] Recalculate totals and taxes on the server.
- [ ] Add idempotent order creation with transactional order/item writes.
- [ ] Add safe public order tracking token and privacy-minimized status response.
- [ ] Test duplicate requests, unavailable products, invalid modifiers, and concurrent orders.

## Phase 4 — Staff operations
- [ ] Implement role-specific navigation and protected views.
- [ ] Complete menu/category CRUD and stock controls.
- [ ] Complete table management and HTTPS QR generation.
- [ ] Complete kitchen order board and valid state transitions.
- [ ] Add cashier cash-collection workflow and reconciliation.
- [ ] Complete owner settings, staff management, and sales reports.
- [ ] Add audit trail for sensitive actions.

## Phase 5 — UPI payment integration
- [ ] Confirm provider, merchant onboarding, supported UPI methods, and webhook documentation.
- [ ] Implement provider adapter and server-side payment intent creation.
- [ ] Verify webhook signature/hash, transaction, amount, currency, and order mapping.
- [ ] Add replay protection, idempotent webhook processing, and reconciliation.
- [ ] Remove all demo secrets and any verification bypass.
- [ ] Test pending, success, failure, cancellation, duplicate callback, and mismatched amount cases in sandbox.
- [ ] Enable live payments only after owner/provider sign-off.

## Phase 6 — QA, security, and production readiness
- [ ] Unit, integration, end-to-end, and security tests.
- [ ] Accessibility and mobile/browser testing.
- [ ] Load and resilience tests for expected restaurant traffic.
- [ ] Dependency and static security review.
- [ ] HTTPS, CSP, CORS, cookies, rate limits, and production config verification.
- [ ] Backup restore and rollback rehearsal.
- [ ] Monitoring, alerts, health checks, privacy notice, and retention policy.
- [ ] Owner acceptance test and operational staff training.

## Phase 7 — Deployment and launch
- [ ] Provision staging and production environments.
- [ ] Configure domain, HTTPS, environment secrets, and managed database.
- [ ] Run migrations and import validated production data.
- [ ] Deploy staging and complete acceptance tests.
- [ ] Deploy production with rollback plan.
- [ ] Verify QR ordering, kitchen flow, cash collection, and UPI sandbox/live flow.
- [ ] Monitor launch and resolve issues.

## Definition of done

A feature is complete only when its UI, server behavior, authorization, validation, persistence, error handling, tests, and documentation are complete. A screen that only looks finished is not a production feature.

## Immediate next task

Complete a route-by-route security inventory and repository hygiene pass before implementing additional business features. The existing server currently contains demo authentication and sensitive routes that require protection; public launch must remain blocked until these are corrected.
