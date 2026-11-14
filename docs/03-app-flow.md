# TableBite — Application Flows

## Guest QR ordering
1. Scan QR containing canonical HTTPS URL, table number, and opaque token.
2. Server resolves token to an active table; invalid/revoked tokens show a safe error.
3. Load restaurant settings and available menu.
4. Browse categories, search, filter dietary preferences, and view product details.
5. Add products with validated portions, add-ons, quantities, and notes.
6. Review cart; client displays an estimate while server remains authoritative.
7. Submit order with idempotency key and table context.
8. Server validates table, products, modifiers, availability, quantities, and calculates totals inside a transaction.
9. Persist order and item snapshots; return order number and public tracking token.
10. Choose UPI payment or cash at counter. UPI intent is created by provider adapter; cash remains unpaid until cashier collection.
11. Kitchen receives new order; staff accept and advance preparation state.
12. Guest tracking view polls/reloads server-confirmed status.
13. Cashier records cash collection or provider webhook confirms UPI.
14. Order is served/completed; table occupancy is recalculated from active orders.

## Staff authentication
1. Staff opens staff portal.
2. Submit email/password over HTTPS.
3. Server rate-limits attempt, verifies password hash, checks active account.
4. Server creates opaque session; only hashed session identifier is stored server-side.
5. Browser receives secure HttpOnly SameSite cookie.
6. Every staff API request validates session and checks role/permission.
7. Logout revokes session and clears cookie; expiry and password reset revoke old sessions.

## Kitchen workflow
1. Kitchen staff signs in and opens KDS.
2. KDS loads orders permitted for kitchen role.
3. New order appears with table, items, modifiers, notes, and elapsed time.
4. Kitchen accepts and moves order to preparing.
5. Kitchen marks order ready; guest tracking updates.
6. Authorized staff marks served/completed according to restaurant workflow.
7. Each transition creates an immutable status event with actor and timestamp.

## Cashier workflow
1. Cashier signs in and views orders requiring payment reconciliation.
2. Cash collection: verify amount received, record amount and cashier identity, then mark paid.
3. UPI: view provider-confirmed payment state and transaction reference; do not manually trust guest screenshots or browser redirects.
4. Exceptions remain pending/failed and are escalated for reconciliation.
5. Every payment action is audited.

## Owner/admin workflow
- Owner configures restaurant identity, tax policy, provider settings, staff accounts, roles, and table registry.
- Admin manages allowed menu/category/table/order operations.
- Reports use server-side date filters and distinguish gross order totals from actually collected revenue.
- Sensitive settings are masked and never returned to unauthorized roles.
