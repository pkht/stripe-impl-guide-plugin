# Product Templates — Sub-section Structure per Stripe Product

Use these as the section blueprints when generating a guide. Expand each sub-section with client-specific content.

---

## Payments — Checkout Sessions & Payment Intents

Use for any online/card-not-present payment collection, whether direct (single account) or within a Connect platform. Present both integration paths and recommend one based on the brief.

### [n]. Payments

#### [n.1] Integration Approach

Open with the Checkout Session vs Payment Intent decision:

| | Stripe Checkout Session | Payment Intent + Elements |
| :---- | :---- | :---- |
| Integration effort | Low — Stripe-hosted page | Higher — custom UI |
| Customisation | Limited (colours, logo, custom fields) | Full control |
| Payment method support | Automatic — Stripe adds new PMs | Must configure and maintain |
| 3DS / SCA | Fully automatic | Must handle `requires_action` / `next_action` |
| Localisation | Built in (30+ languages) | Client responsibility |
| Best for | Speed to market, conversion optimisation | Brand-critical or bespoke checkout flows |

State the recommended approach for this client and why.

---

#### [n.2] Stripe Checkout Sessions

##### Creating a Checkout Session (server-side)

Key parameters:
- `mode` — `payment` (one-time), `subscription` (recurring), or `setup` (save without charging)
- `line_items` — price ID(s) or inline price data
- `success_url` / `cancel_url` — where to redirect after payment or cancellation
- `customer` — attach to an existing Customer for saved payment method support
- `payment_intent_data.setup_future_usage` — `'on_session'` or `'off_session'` to save the card after payment
- `metadata` — include order ID, location, or any reconciliation data
- `payment_method_types` — leave unset to use Stripe's automatic payment method selection, or specify explicitly

Code example: create session, return `url`, redirect client.

##### Fulfilling the Order

- Do not fulfil on redirect to `success_url` alone — the customer could arrive there without paying (e.g. by navigating directly)
- Listen for `checkout.session.completed` webhook to confirm payment before fulfilling
- Check `payment_status` on the session object: `paid` | `unpaid` | `no_payment_required`

##### Connect-specific parameters *(include if Connect is also in scope)*
- `payment_intent_data.on_behalf_of` — run the charge on a connected account
- `payment_intent_data.application_fee_amount` — platform fee in minor currency units
- `payment_intent_data.transfer_data.destination` — where to transfer funds

---

#### [n.3] Payment Intents + Stripe Elements

##### Server-side: Creating a Payment Intent

Key parameters:
- `amount`, `currency`
- `capture_method` — `automatic` (default) or `manual` (auth & capture)
- `customer` — for saved payment methods
- `setup_future_usage` — `'on_session'` or `'off_session'`
- `metadata` — for reconciliation

Code example: create PaymentIntent, return `client_secret` to frontend.

##### Auth & Capture

Use `capture_method: manual` when the client needs to confirm availability (e.g. inventory check, hotel hold) before capturing funds. Stripe holds the authorisation for up to 7 days.

- Capture: `PaymentIntent.capture(id)` — captures full amount by default
- Partial capture: pass `amount_to_capture`
- Cancel (release hold): `PaymentIntent.cancel(id)`

##### Client-side: Stripe Elements

- Load `stripe.js` on every page in the checkout flow (fraud signal collection)
- Initialise: `const stripe = Stripe(publishableKey)`
- Mount the Payment Element using the `client_secret`
- Call `stripe.confirmPayment()` with `return_url`
- Handle `requires_action` / redirect-based payment methods via `next_action`

##### Connect-specific parameters *(include if Connect is also in scope)*
- `on_behalf_of` — connected account ID
- `transfer_data.destination` — where to transfer funds
- `application_fee_amount` — platform fee

---

#### [n.4] Saving Payment Methods

- Create a Customer object and store the `customer_id` in your database
- Use `setup_future_usage` on the PaymentIntent or Checkout Session to attach the payment method to the Customer after payment
- For saving without an immediate charge: use a SetupIntent instead
- `'on_session'` — customer is present; optimised for low-friction re-use
- `'off_session'` — card will be charged later without the customer present (e.g. subscriptions, invoices)
- List saved payment methods via `stripe.paymentMethods.list({ customer, type: 'card' })`

---

#### [n.5] Refunds

- Issue refunds via the Refunds API or Stripe Dashboard
- Reference the `payment_intent` ID or `charge` ID
- Full refund: omit `amount`; partial refund: pass `amount` in minor currency units
- Multiple partial refunds allowed up to the original charge total
- Webhook `charge.refund.updated` — handle `status: failed` (rare but must be caught)
- Connect: refunds debit the connected account balance; the platform's application fee is returned proportionally by default

---

#### [n.6] Disputes

- Any charge can be disputed; have a process in place before going live
- Webhook `charge.dispute.created` — trigger internal notification and begin evidence collection
- Evidence window: typically 7–21 days depending on card network
- Submit evidence via the Disputes API or Dashboard
- `charge.dispute.closed` — outcome: `won`, `lost`, or `needs_response`

---

#### [n.7] Financial Reconciliation

- Include `metadata` on every PaymentIntent / Checkout Session with internal reference IDs (order ID, location, customer ref)
- Stripe Financial Reports: Balance report, Payout reconciliation report (recommended)
- Reporting API: listen for `reporting.report_type.updated`, then download via API
- Dashboard exports for ad-hoc queries
- Third-party connectors: NetSuite, QuickBooks (via Stripe Apps & Extensions)

---

## Stripe Connect

### [n]. Stripe Connect

#### [n.1] Connect Configuration
- Account type (Custom / Express / Standard)
- Controller settings (payouts, dashboard, losses)
- Country setting, external account restrictions
- Statement descriptor
- Configuration table

#### [n.2] Fund Flow
Mermaid flowchart showing:
- Customer → PaymentIntent → Connected Account balance
- Application fee → Platform account
- Payout → Connected account's bank account

#### [n.3] Onboarding

##### Seller Authentication
How the platform authenticates its sellers before creating/fetching their Stripe connected account. Cover the recommended patterns (JWT, OAuth, SSO, deep link).

##### Account State Machine
Mermaid stateDiagram showing the states a connected account moves through:
- Created → OnboardingRequired → PendingVerification → Ready → Restricted

Include the backend state derivation logic (how to read `requirements.entries`, `capabilities`, etc.)

##### Step-by-step onboarding flow
Number the steps:
1. Validate & Register (backend verifies account exists, creates if new)
2. Poll Account State (GET endpoint that re-polls Stripe)
3. Serve Embedded Onboarding Session (AccountSession creation, client secret)
4. Keep State in Sync (webhooks for account updates)

For each step: what the backend does, what the mobile/web app does, code example.

##### Onboarding Endpoint Reference
Table: Method | Path | Auth | Description

##### Onboarding Troubleshooting
Table of common issues: Symptom | Likely cause | Fix

#### [n.4] Monetisation

##### Fee Summary
Table: Fee type | Amount | Who pays | Notes

##### Configuring the Application Fee
Code example: creating a PaymentIntent with `application_fee_amount` or `transfer_data`.

##### Payment Fee Flow
Mermaid sequence diagram: Customer → Platform → Connected Account, showing fee split.

#### [n.5] Payouts

- Payout schedule configuration (daily/weekly/manual)
- Platform control over connected account schedule changes
- Instant Payouts (if applicable) — eligibility, fee, API call

#### [n.6] Connect Dashboards
- What connected accounts can see (none / express / full)
- Embedded component access via Account Sessions
- Livemode vs testmode considerations

---

## Stripe Terminal

### [n]. Payments — Stripe Terminal

#### [n.1] Integration Design

##### SDK / Integration Shape
State which SDK the client will use (from `product-detection.md` reader matrix) and why. Include a brief comparison table if multiple options were considered.

Always set `AppInfo` when initialising the SDK — Stripe uses this for observability and partner tracking:
```
AppInfo: { name: "<CompanyName>Middleware", version: "<version>", url: "<url>" }
```

##### Authentication
- Standard integrations: Stripe secret key on the backend only
- POS partner/app integrations: Restricted API Keys (RAKs) — scoped permissions, one key per app install. Note if gated access is needed for Connect orchestration permissions.

#### [n.2] Terminal Registration

##### Locations
- What a Terminal Location is and why it's needed
- Code: create/retrieve location (include address — required)
- How locations map to the client's business structure (one per physical site)
- Dashboard vs API registration

##### Connection Token
- What the connection token is (ephemeral, single-use, grants SDK access to a Stripe account)
- Backend endpoint that issues it
- Code: `ConnectionToken.create()` — include `Stripe-Account` header if using Connect
- SDK calls this automatically whenever it needs a fresh token

##### Reader Discovery & Connection
- SDK `discoverReaders()` call with discovery method (internet, bluetooth, USB)
- `connectReader()` flow
- Reader registration via Dashboard or API (pairing code → Reader object)

#### [n.3] Payments

##### Create a PaymentIntent
- Parameters: `amount`, `currency`, `payment_method_types: ["card_present"]`, `capture_method: "manual"`, `on_behalf_of` + `transfer_data` (if Connect)
- Note: `capture_method: manual` is required for Terminal — auto-capture is not supported for card-present
- Client-side vs server-side creation — note that **server-side creation does not support offline mode**
- Code example (backend)

##### Collect Payment Method
- SDK `collectPaymentMethod(paymentIntent)` call
- Handling `requires_payment_method` after a decline — must collect again before confirming

##### Confirm the Payment
- SDK `confirmPaymentIntent()` call
- Mobile wallet payments (Apple Pay, Google Pay) — treated as contactless EMV, no extra parameters needed

##### Capture a PaymentIntent
- When to capture (after successful reader interaction)
- Code: `PaymentIntent.capture(id)`
- Auto-capture option (if applicable)

##### Terminal Payment Endpoint Reference
Table: Method | Path | Auth | Description

##### Required Stripe API Calls
Sequence diagram: POS App → Backend → Stripe API → Reader, showing the full payment lifecycle

#### [n.4] Offline Payments *(include if offline mode is a requirement)*

- Prerequisites: reader must have been online in last 30 days; reader and POS on same local WiFi
- Configure `OfflineBehavior` on the PaymentIntent:

| Value | Behaviour |
| :---- | :---- |
| `RequireOnline` | Throws if device is offline |
| `PreferOnline` | Allows offline if needed, prefers online |
| `ForceOffline` | Always offline — for latency-critical use cases |

- Enable offline mode at the Location level via the API (Dashboard not supported)
- Implement `IOfflineListener` to handle online↔offline transitions and forwarded payment status
- Offline maximum: $10,000 USD per transaction (Stripe-enforced)
- Risk management: configure per-transaction and per-reader cumulative limits
- Payment forwarding: automatic when connectivity restored — monitor via `OfflinePaymentsCount` and `OfflineStatus`
- Receipts: retrieve from `paymentIntent.OfflineDetails.OfflineCardPresentDetails` when offline

#### [n.5] Advanced Payment Flows *(include relevant sub-sections only)*

##### Tap to Pay
- Supported SDKs: iOS, Android, React Native only
- Payment flow identical to card-present — no extra parameters
- Note: Tap to Pay is not available offline in SCA-regulated markets

##### MOTO (Mail Order / Telephone Order)
- Gated feature — requires Stripe support approval
- Payment flow differs from card-present: no reader interaction, card details entered manually

##### Dynamic Currency Conversion
- Pass `requestDynamicCurrencyConversion` in `CollectConfiguration`
- Cardholder sees price in their home currency at point of interaction

##### Unlinked Refunds
- Gated feature — requires Stripe support approval
- Use cases: refund to different card, refund for payment from another PSP
- Flow: create Customer → collect card via SetupIntent → refund to PaymentMethod

#### [n.6] Refunds

- Standard refunds: Refund API from backend (not via reader), using `charge_id` or `payment_intent_id`
- Partial refunds: pass `amount`
- **Interac (Canada):** must be processed in-person via reader — cannot use API-only refund
- Cancelling a PaymentIntent: available before capture; releases held funds
- Cancelling a Terminal action: `cancelReaderAction()` (SDK) or `cancel_action` API (server-driven)
- Key behaviour: refund routing for Connect (refunds flow back through the connected account)

#### [n.7] Disputes

- Platform dispute handling steps
- Notifying connected accounts via webhooks
- Evidence submission window and process

---

## Stripe Billing

### [n]. Stripe Billing

#### [n.1] Billing Configuration
- Products and prices setup
- Billing cycle, proration, grace periods
- Tax configuration (Stripe Tax vs manual)
- Payment collection method (charge automatically / send invoice / charge then invoice)
- Customer portal configuration

#### [n.2] Product & Price Catalogue
Table of the client's products/prices (from brief or MCP data):

| Product | Price ID | Amount | Interval | Model |
| :---- | :---- | :---- | :---- | :---- |

Code: creating products and prices via API (if dynamic catalogue)

#### [n.3] Customer Management
- Creating/retrieving customers (`Customer.create`)
- Attaching payment methods
- Default payment method for subscriptions

#### [n.4] Subscriptions

##### Creating a Subscription
Code: `Subscription.create()` with items, default_payment_method, trial, metadata

##### Subscription Lifecycle
Mermaid stateDiagram: trialing → active → past_due → canceled / unpaid

##### Upgrades & Downgrades
- Proration behaviour
- Code: `Subscription.update()` with `proration_behavior`

##### Cancellation
- Immediate vs end-of-period
- Code example

#### [n.5] Invoices
- Automatic vs manual invoices
- Invoice finalization and payment collection
- Invoice PDF access
- Key webhook events: `invoice.payment_succeeded`, `invoice.payment_failed`, `invoice.finalized`

#### [n.6] Dunning & Retries
- Smart Retries configuration
- Retry schedule
- What happens on final failure (subscription cancellation vs unpaid)

#### [n.7] Customer Portal
- What end users can do (manage payment methods, cancel, view invoices)
- Backend: creating a portal session
- Redirect flow

#### [n.8] Webhook Events Reference
Table: Event | When it fires | Required action

---

## Stripe Radar

### [n]. Fraud & Risk — Stripe Radar

#### [n.1] Overview

Brief description of how Radar works — machine learning scoring on every charge across Stripe's network, real-time decision at point of payment. Clarify which tier is in scope:

| Tier | What's included | Integration needed |
| :---- | :---- | :---- |
| Radar (default) | ML scoring, default block/allow/review rules | None — active on all accounts |
| Radar for Teams | Custom rules, allow/block lists, manual review queue, rule testing | Dashboard configuration; confirm if gated |

#### [n.2] Default Rules

- Overview of built-in rules (block if CVC fails, block if card is on Stripe's fraud network, etc.)
- Recommended baseline: always block if CVC verification fails
- Where to view and configure in Dashboard: Settings → Radar Rules

#### [n.3] Custom Rules *(Radar for Teams only)*

- Rule syntax: `if [attribute] [operator] [value] then [block | review | allow]`
- Common rule patterns:
  - Block by country: `if :ip_country: = 'XX'`
  - Block above amount threshold: `if :amount_in_gbp: > 500`
  - Allow known customers: `if :customer_id: in @trusted_customers`
- How to test rules against historical payments in the Dashboard before enabling
- Allow lists and block lists — managing via Dashboard or API

#### [n.4] 3D Secure (3DS / SCA)

- When Radar triggers 3DS: based on risk score, SCA regulatory requirements (EEA/UK), or explicit rules
- Payment Intent flow when `requires_action` is returned:
  - Checkout Session: Stripe handles 3DS automatically — no client-side code needed
  - Payment Intent + Elements: call `stripe.handleNextAction(clientSecret)` or use `return_url` with redirect
- `payment_intent.payment_failed` with `last_payment_error.code: authentication_required` — how to handle
- Requesting 3DS explicitly: set `payment_method_options.card.request_three_d_secure: 'any'` on the PaymentIntent

#### [n.5] Early Fraud Warnings

- Stripe issues an `radar.early_fraud_warning.created` webhook when a card network signals likely fraud
- ~82% of early fraud warnings escalate to disputes
- Recommended action: automatically issue a full refund on receipt of this event to avoid the dispute fee
- Code: listen for webhook → retrieve charge → issue refund if not already refunded

#### [n.6] Dispute Prevention

- Align statement descriptor with brand name customers recognise (reduces "I don't recognise this" disputes)
- Send payment confirmation emails with clear charge description and customer support contact
- For high-risk categories: consider requiring 3DS for all transactions
- Monitor dispute rate in Dashboard — Stripe may restrict the account if rate exceeds network thresholds (~1%)

---

## Reporting

### [n]. Reporting

#### [n.1] Approach

Cover which reporting mechanisms are available and appropriate for the client:

| Mechanism | Best for | Integration required |
| :---- | :---- | :---- |
| Stripe Dashboard | Manual review, one-off exports | None |
| Financial Reports (pre-built) | Closing the books, payout reconciliation | None (or webhook to automate) |
| Reporting API | Automated ingestion into accounting/BI systems | Yes — poll or webhook-triggered |
| Embedded components (web) | Exposing reports to end users within the platform UI | Account Session + component |
| Embedded components (mobile) | Limited — payments list and payouts only (Preview) | Account Session + mobile SDK |
| Sigma | Ad-hoc SQL analytics, custom reports | Dashboard access (paid add-on) |
| Revenue Recognition | Accrual accounting, deferred revenue | Stripe Revenue Recognition (paid add-on) |

State which mechanisms are in scope for this client.

#### [n.2] Financial Reports (Reporting API)

Available report types:

| Report type | `report_type` value | What it covers |
| :---- | :---- | :---- |
| Balance change from activity | `balance_change_from_activity.itemized.3` | Every transaction affecting the Stripe balance |
| Payouts | `payouts.itemized.3` | Breakdown of what's included in each payout |
| Payout reconciliation | `payout_reconciliation.itemized.3` | Match payouts to their underlying charges |
| Connect platform earnings | `connect_platform_earnings.itemized.1` | Application fees and Connect earnings |
| IC+ payment fees | `payment_charges.itemized.2` | Interchange-plus fee breakdown per charge |

**Generating a report:**
1. Create a Report Run: `POST /v1/reporting/report_runs` with `report_type`, `parameters.interval_start`, `parameters.interval_end`
2. Poll or wait for `reporting.report_run.succeeded` webhook
3. Retrieve the download URL from `report_run.result.url`
4. Download via authenticated GET request

Sequence diagram: Backend → create ReportRun → webhook fires → download URL → parse CSV

Key notes:
- Reports use UTC timestamps; data is typically available with a 1–2 day lag
- Specify `columns` parameter to limit output to required fields
- For Connect: add `parameters.connected_account` to scope to a specific connected account

#### [n.3] Embedded Reporting Components *(include if applicable)*

**Web:**
- Documents component — statements, tax forms
- Account management — payments list with export

**Mobile (iOS/Android):**
- Payments component (Preview) — list of payments with export
- Payouts component (Preview) — balance and payout history
- No other reporting components available on mobile

Setup: create an Account Session with the required component enabled, return `client_secret` to the frontend/app, initialise the component with the session.

#### [n.4] Specific Reports *(add one sub-section per report the client needs)*

For each specific report, document:
- Report type and variant (`report_type` value)
- Parameters (date range, currency, account scope)
- Data availability lag
- Key columns and their meaning
- Sequence diagram if the report requires a multi-step backend flow
- Any known gotchas (e.g. one row per fee component, not one row per charge)

---

## Webhooks (cross-cutting)

Include a dedicated Webhooks section if multiple products share webhook infrastructure, or inline webhook tables within each product section for simpler guides.

### Webhook Setup
- Registering the endpoint in the Stripe Dashboard or via API
- Signature verification (always required — show `Webhook.constructEvent`)
- Idempotency handling (store processed event IDs)

### Webhook Events Reference
Table per product area:

| Event | Product | Trigger | Required action |
| :---- | :---- | :---- | :---- |
| `account.updated` | Connect | Connected account fields change | Re-derive account state, update local DB |
| `v2.core.account.person.updated` | Connect (v2) | Identity document reviewed | Re-derive account state |
| `payment_intent.succeeded` | Terminal / Payments | Payment captured | Record transaction, trigger payout logic |
| `charge.dispute.created` | Terminal / Payments | Dispute opened | Notify connected account, queue evidence |
| `invoice.payment_succeeded` | Billing | Invoice paid | Provision access, send receipt |
| `invoice.payment_failed` | Billing | Payment failed | Trigger retry flow, notify customer |
| `customer.subscription.deleted` | Billing | Subscription cancelled | Revoke access |
