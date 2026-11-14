# TableBite — UI/UX Design Brief

## Design principles
- Mobile-first guest experience; tablet/desktop optimized staff dashboards.
- Clear, calm visual hierarchy suitable for busy restaurant service.
- Fast primary actions, readable order details, accessible contrast, keyboard support, and visible focus states.
- Responsive layouts with loading, empty, offline, validation, and error states.
- Use semantic HTML and avoid relying on color alone for order/payment state.

## Guest experience
- Branded restaurant header and validated table indicator.
- Search, category navigation, dietary filters, product cards, sold-out state.
- Accessible customization drawer for portion/add-ons and cooking notes.
- Persistent cart summary and transparent tax/total breakdown.
- Checkout with name/phone as configured, explicit UPI or cash choice, clear payment pending/confirmed state.
- Order tracking timeline with order number, table, items, and status.
- Never show a false “paid” state before server/provider verification.

## Owner/admin portal
- Secure login and role-aware navigation.
- Overview: revenue collected, orders, average order value, active kitchen orders, payment exceptions.
- Menu CMS: categories, product details, pricing, modifiers, availability, image.
- Table registry: table number, capacity, QR preview/download, token rotation.
- Orders and receipts with filters.
- Staff and role management, sensitive settings, audit history.
- Reports with date range and export.

## Kitchen display
- High-visibility columns for New, Preparing, Ready, and Completed.
- Order cards show table, elapsed time, quantities, modifiers, item notes, and timestamps.
- Large touch-friendly actions and optional audible new-order alert.
- Confirm consequential actions and show update failures/retry state.

## Cashier
- Payment queue separated into pending UPI, cash due, and reconciled.
- Clear amount due, order number, table, and payment method.
- Cash collection confirmation requires explicit action and records staff identity.
- UPI state is sourced from verified backend provider state.

## Accessibility and usability
- WCAG 2.2 AA target for core flows.
- Labels and instructions for form fields; error messages associated with fields.
- Focus management for dialogs/drawers and escape-to-close behavior.
- Touch targets sized for mobile/tablet use.
- Test at narrow mobile, tablet landscape, and desktop breakpoints.
