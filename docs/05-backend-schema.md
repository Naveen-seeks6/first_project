# TableBite — Backend Data Schema Proposal

The production target is PostgreSQL. This is a logical schema proposal; exact SQL migrations, constraints, indexes, and migration scripts must be reviewed and tested before use.

## Identity and access
### staff_users
- id UUID primary key
- email normalized unique
- display_name
- password_hash
- role enum: owner, admin, kitchen, cashier
- is_active boolean
- created_at, updated_at

### staff_sessions
- id UUID primary key
- staff_user_id references staff_users
- token_hash unique
- expires_at, revoked_at, created_at

## Catalog
### categories
- id UUID primary key
- name, icon, display_order, is_active
- created_at, updated_at

### products
- id UUID primary key
- category_id references categories
- name, description
- price_minor BIGINT (paise)
- image_url
- is_veg, is_spicy, is_bestseller, is_available
- created_at, updated_at

### product_portions / product_addons
- id UUID primary key
- product_id references products
- name
- price_minor BIGINT
- is_active

## Tables and QR identity
### restaurant_tables
- id UUID primary key
- table_number unique
- qr_token_hash unique (never store raw token)
- capacity
- status
- created_at, updated_at

## Orders
### orders
- id UUID primary key
- order_number unique
- public_tracking_token_hash unique
- table_id references restaurant_tables
- customer_name, customer_phone (nullable/minimized)
- status
- payment_method: upi or cash_counter
- payment_status: pending, paid, failed, cancelled, refunded
- subtotal_minor, tax_minor, total_minor BIGINT
- idempotency_key unique
- created_at, updated_at

### order_items
- id UUID primary key
- order_id references orders on delete cascade
- product_id nullable reference products
- item_name_snapshot
- unit_price_minor BIGINT
- quantity with positive bounded check
- selected_portion_snapshot JSONB
- selected_addons_snapshot JSONB
- item_total_minor BIGINT
- special_note

### order_status_events
- id UUID primary key
- order_id references orders
- from_status, to_status
- actor_staff_id nullable references staff_users
- created_at

## Payment and audit
### payment_transactions
- id UUID primary key
- order_id references orders
- provider
- provider_transaction_id
- provider_event_id unique where present
- amount_minor BIGINT, currency
- status
- verified_at, created_at

### cash_collections
- id UUID primary key
- order_id references orders
- amount_minor BIGINT
- cashier_staff_id references staff_users
- collected_at
- note

### audit_logs
- id UUID primary key
- actor_staff_id nullable references staff_users
- action, resource_type, resource_id
- metadata_redacted JSONB
- created_at

### restaurant_settings
- key primary key
- value JSONB or text
- updated_by references staff_users
- updated_at

### idempotency_records
- key primary key
- request_hash
- response_reference
- created_at

## Integrity rules
- Use integer paise for all monetary values.
- Product and modifier prices must be non-negative.
- Quantity must be a positive bounded integer.
- Order and line-item creation must be one transaction.
- Server computes all monetary totals.
- Preserve item name/price/modifier snapshots in historical orders.
- Enforce unique provider transaction/event identifiers and idempotent webhook handling.
- Every status and payment transition must be recorded.
- Use database constraints for uniqueness, foreign keys, and valid states in addition to application validation.
- Never store raw staff session tokens, QR tokens, or public tracking tokens; store secure hashes.
- Avoid returning internal IDs, staff details, payment secrets, or customer phone numbers in public tracking responses.
