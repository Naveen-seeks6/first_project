# TableBite — Product Requirements Document (PRD)

**Status:** Approved for technical design  
**Product:** TableBite  
**Business model:** Single restaurant  
**Initial payment methods:** UPI and cash at counter  
**Access model:** Owner, administrator, kitchen staff, cashier  
**Target:** Production deployment  
**Repository:** `Naveen-seeks6/first_project`

## 1. Product vision

TableBite is a mobile-first restaurant ordering and operations application. Guests scan a QR code associated with their table, browse the menu, customize items, place orders, choose UPI or cash at counter, and follow order progress. Authorized restaurant staff use role-appropriate dashboards to manage orders, preparation, payments, menu availability, tables, staff, settings, and reports.

The first release serves one restaurant. Data and service boundaries should avoid unnecessary assumptions that would prevent a future multi-tenant edition, but multi-restaurant onboarding and tenant billing are explicitly out of scope.

## 2. Goals and success criteria

- A guest can place an order from a valid table QR on a mobile browser without installing an app.
- The server is authoritative for menu availability, prices, taxes, totals, order status, and payment state.
- New orders appear in the kitchen workflow promptly and status changes are visible to guests.
- Staff can only access actions permitted by their assigned role.
- Cash and UPI orders have clear, auditable payment states.
- Restaurant operations can recover from a process restart without losing committed orders.
- Production deployment has HTTPS, backups, monitoring, secret management, and documented recovery procedures.

## 3. Users and roles

### Guest
- Open the menu using a table QR.
- Browse/search/filter categories and dishes.
- Select portions/add-ons and enter item notes.
- Create an order and receive a non-guessable order tracking reference.
- View order progress and payment state.

### Owner
- Full access to restaurant settings, staff, reports, menu, tables, orders, and payment reconciliation.
- Manage administrator accounts and sensitive configuration.

### Administrator
- Manage menu, categories, tables, orders, and operational settings according to permissions granted by the owner.
- View operational reports.
- Cannot change owner credentials or access secrets unless explicitly authorized.

### Kitchen staff
- View incoming orders and item instructions.
- Move orders through accepted/preparing/ready/served states.
- Cannot change prices, settings, staff roles, or payment state.

### Cashier
- View orders and bill details.
- Record cash received and reconcile UPI payments using authorized evidence/provider status.
- Cannot alter menu prices or kitchen preparation state unless separately permitted.

## 4. Functional requirements

### Customer ordering
- QR links bind to a known table using a revocable token; table number alone is not proof of a valid QR.
- Menu displays categories, product details, dietary labels, availability, portions, and add-ons.
- Cart supports quantity changes, item removal, customizations, and clear price breakdown.
- Server recalculates every line item and tax; client-submitted prices are never trusted.
- Validate quantity bounds, product availability, portions, add-ons, table token, and request size.
- Prevent duplicate orders through an idempotency key.
- Return a stable order reference and status lookup capability that does not expose other customers' personal data.

### Order lifecycle
- Initial state: `new`.
- Operational states: `accepted`, `in_kitchen`, `ready`, `served`, `completed`, `cancelled`.
- Enforce allowed state transitions on the server and record actor/time for every transition.
- Keep order and line-item snapshots so later menu edits do not rewrite historical receipts.

### Payments
- Support UPI checkout through an approved payment provider and cash at counter.
- UPI starts as `pending`; mark paid only after verified provider callback/webhook or server-to-server status verification.
- Verify provider signature/hash, transaction identity, amount, currency, merchant account, and order mapping; make webhook handling idempotent.
- Never expose merchant salt, secret keys, or service-role keys to the browser.
- Cash orders remain `pending` until an authorized cashier records collection; record who collected it and when.
- Support failed, cancelled, pending, paid, and refunded states as applicable.
- Do not treat a user clicking “I paid” or returning to a success URL as proof of payment.

### Staff and security
- Individual staff accounts with secure password hashing and server-managed sessions.
- Role-based authorization enforced by middleware on every protected API route.
- Login throttling, session expiry/revocation, secure cookies, CSRF protection for cookie-authenticated state-changing requests, and audit logging.
- No demo credentials, PIN bypasses, local-token fallbacks, or default production passwords.

### Admin and operations
- Dashboard metrics for revenue, order count, average order value, active orders, and popular dishes.
- CRUD for categories and products; toggle availability.
- Table creation, capacity/status management, QR generation, token rotation/revocation.
- Staff account and role management.
- Restaurant settings for name, contact details, tax configuration, currency, and payment provider configuration.
- Sales reports with date filters and export capability.

### Kitchen
- Live or near-live order board grouped by status.
- Clear quantities, modifiers, special notes, elapsed time, and table number.
- Sound alert for newly received orders, with user-controlled mute.
- State transitions validated by server permissions.

## 5. Non-functional requirements

- Mobile-first, accessible interface with keyboard navigation and clear focus states.
- HTTPS-only production traffic.
- Server-side validation and parameterized database queries.
- Least-privilege access and no secrets committed to source control.
- Persistent managed database with automated backups and tested restore process.
- Structured logs, health endpoint, error monitoring, and operational alerts.
- Reasonable API rate limits and payload limits.
- Graceful handling of network interruptions and payment-provider delays.
- Documented deployment, rollback, backup, and incident procedures.

## 6. Out of scope for first release

- Multiple restaurant tenants and SaaS subscriptions.
- Native iOS/Android apps.
- Delivery fleet management.
- Loyalty points, coupons, inventory purchasing, payroll, and accounting integrations.
- Card payments unless separately approved and integrated with the selected provider.

## 7. Acceptance criteria

1. A valid table QR opens the correct restaurant/table context; invalid or revoked tokens cannot place orders.
2. Guest order totals match server-side calculations, including configured tax.
3. Repeated submission with the same idempotency key creates only one order.
4. Orders and line items survive application restart.
5. Kitchen and customer status views show server-confirmed order state.
6. UPI orders cannot become paid without verified provider evidence.
7. Cash orders cannot become paid except through an authorized cashier/owner action.
8. Unauthorized roles receive 401/403 responses for protected endpoints.
9. No secrets or demo credentials are present in the production bundle or repository.
10. Automated tests cover authentication, authorization, order creation, pricing, transitions, and payment callbacks.
11. Production HTTPS, backups, monitoring, and rollback are verified before launch.

## 8. Assumptions requiring owner confirmation before go-live

- Legal restaurant name, address, contact details, GST registration/tax treatment, and receipt wording.
- Exact payment provider and live merchant onboarding.
- UPI provider support and webhook configuration.
- Staff roster and initial role assignments.
- Hosting region, domain name, data retention, and privacy policy.
- Whether customer phone number is mandatory or optional.

Tax configuration must be reviewed by the restaurant's accountant; the existing 5% default is not treated as legal/tax advice.
