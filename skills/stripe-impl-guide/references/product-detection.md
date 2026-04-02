# Product Detection — Identifying Stripe Products from a Brief

Use this reference to identify which Stripe products are in scope from a client brief. Look for explicit names first, then infer from described use cases.

---

## Stripe Connect

**Explicit signals:** "Connect", "connected accounts", "marketplace", "platform", "sub-merchants", "sellers", "split payments", "application fee", "payout to third parties", "white-label payments"

**Use case signals:**
- Client wants to enable *other businesses* to accept payments through their platform
- Client needs to pay out funds to third parties (creators, vendors, contractors, festivals, merchants)
- Client wants to charge a fee on transactions processed by others
- "Buy-rate" or "blended pricing" model mentioned

**Sub-type determination:**

| Signal | Type |
| :---- | :---- |
| "buy-rate", "IC+", "wholesale Stripe pricing" | Buy-rate Connect (platform sees raw Stripe fees) |
| "blended", "fixed rate", "we charge sellers X%" | Blended / standard Connect |
| Connected accounts have their own Stripe dashboard access | Express or Standard accounts |
| Connected accounts have no Stripe dashboard ("white label") | Custom accounts (`dashboard: none`) |
| Platform manages all payouts | Controller-based payouts |
| Connected accounts manage own payouts | Self-managed payouts |

---

## Payments — Checkout Sessions & Payment Intents

Covers online/card-not-present payment collection. Applies to both direct (single-account) integrations and Connect platforms collecting payments on behalf of connected accounts.

**Explicit signals:** "online payments", "eCommerce", "online store", "Checkout", "Stripe Checkout", "Checkout Session", "Payment Intent", "PaymentIntent", "Elements", "card payments", "accept payments online"

**Use case signals:**
- Client needs to collect card or wallet payments online
- Checkout flow on a website or mobile app
- Auth & capture for order fulfilment (check inventory before capture)
- Saving cards for returning customers
- One-time payments, donations, or order payments

**Integration approach — Checkout Session vs Payment Intent:**

| Signal | Approach |
| :---- | :---- |
| "low effort", "hosted page", "fast to market", no custom checkout UI | Stripe Checkout Session (hosted) |
| Custom checkout UI, full brand control, bespoke flow | Payment Intent + Stripe Elements (custom) |
| "subscriptions" also in scope | Billing — use Checkout Session or Billing API |
| "save cards for later", "returning customers" | Customer object + `setup_future_usage` on either approach |
| Connect platform collecting on behalf of connected accounts | Payment Intent with `on_behalf_of` + `transfer_data` or `application_fee_amount` |

**Connect context:** When used within a Connect integration, both Checkout Sessions and Payment Intents support `on_behalf_of` (to run the charge on the connected account) and `application_fee_amount` / `transfer_data` (to route funds). Cover these parameters within the Connect section of the guide, or cross-reference from the Payments section.

**Note:** Radar is relevant for any online payment integration — include it unless the client explicitly opts out.

---

## Stripe Terminal

**Explicit signals:** "Terminal", "tap to pay", "tap-to-pay", "in-person payments", "card reader", "POS", "point of sale", "BBPOS", "Stripe Reader", "S700", "WisePOS"

**Use case signals:**
- Physical retail payments
- Mobile POS application
- Staff devices used to collect payment from customers
- "Connection token", "reader registration", "terminal location"
- "offline payments", "store and forward"

**SDK selection** — determine the right integration shape from the brief:

| Signal | Recommended integration |
| :---- | :---- |
| Mobile app (iOS/Android/React Native) | Mobile SDK — supports all readers + Tap to Pay + offline |
| Web-based POS | JavaScript SDK |
| Desktop POS (.NET/Java) | Desktop SDK — supports S700, S710, WisePOS E, Verifone |
| Server-managed reader, no client SDK | Server-Driven integration — smart readers only, no offline |
| Offline payments required | Mobile SDK or Desktop Java SDK (not JS or Server-Driven) |

**Reader hardware selection:**

| Reader | Mobile SDK | JS SDK | Desktop SDK | Server-Driven | Tap to Pay | Offline |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| Stripe Reader S700 | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ |
| Stripe Reader S710 | ✅ | ❌ | ✅ | ✅ | ❌ | ✅ |
| BBPOS WisePOS E | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ |
| BBPOS WisePad 3 | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Stripe Reader M2 | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Tap to Pay (device) | ✅ iOS/Android | ❌ | ❌ | ❌ | ✅ | ❌ |

**Platform context:** Terminal is almost always combined with Connect when the merchants are connected accounts running their own devices. For single-merchant direct integrations, Terminal runs on the merchant's own Stripe account.

---

## Stripe Billing

**Explicit signals:** "Billing", "subscriptions", "invoicing", "recurring payments", "usage-based billing", "metered billing", "SaaS pricing", "plans", "price tiers"

**Use case signals:**
- Client bills their customers on a recurring schedule
- Multiple pricing tiers or usage-based models
- Invoice generation and delivery
- Dunning / retry logic for failed payments
- Migration from another billing system (Chargebee, Recurly, PayPal, etc.)
- "Revenue recognition", "prorations", "upgrades/downgrades"

**Sub-type determination:**

| Signal | Sub-type |
| :---- | :---- |
| Fixed monthly/annual fee | Flat-rate subscription |
| Based on units consumed | Metered / usage-based billing |
| Seat-based pricing | Per-seat subscription |
| Complex custom rules | Custom billing with Billing API + webhooks |
| B2B invoicing with NET30/60 | Invoice-based collection |

---

## Stripe Radar

**Explicit signals:** "Radar", "fraud", "fraud rules", "fraud prevention", "3DS", "3D Secure", "dispute rate", "chargeback", "block certain cards", "early fraud warning"

**Use case signals:** Client has elevated fraud risk, needs custom block/allow rules, wants to tune 3DS triggers, or has dispute rate concerns.

**Default vs Radar for Teams:**
- Default Radar rules (no integration needed) — active on all Stripe accounts; suitable for most new integrations
- Radar for Teams — needed when client requires custom rules, allow/block lists, manual review queues, or rule testing. Confirm if gated access is required.

**Note:** Radar is implicitly in scope for any integration that accepts card payments online — always include at minimum a brief section on default rules and early fraud warnings, even if not explicitly requested.

---

## Reporting

**Explicit signals:** "reporting", "reconciliation", "financial reports", "close the books", "payout reconciliation", "IC+ fees", "Sigma", "revenue recognition", "data export", "accounting integration"

**Use case signals:**
- Client needs to reconcile Stripe payouts against internal records
- Finance team needs automated transaction data exports
- Client wants to surface transaction history to end users within their platform
- Specific report types mentioned (IC+ fee breakdown, balance history, etc.)
- Integration with accounting system (NetSuite, QuickBooks, AccountsIQ, etc.)

**Mechanism selection:**

| Signal | Mechanism |
| :---- | :---- |
| "no code", "finance team manages it" | Stripe Dashboard + Financial Reports |
| "automate", "ingest into our system", API-driven | Reporting API (ReportRuns) |
| "show transactions to merchants/sellers" | Embedded components (web: full suite; mobile: payments + payouts only) |
| "SQL", "custom queries", "analytics" | Sigma |
| "deferred revenue", "accrual accounting" | Revenue Recognition |
| "IC+ fee breakdown per transaction" | `payment_charges.itemized` report type |

---

## Stripe Issuing

**Explicit signals:** "Issuing", "virtual cards", "physical cards", "expense cards", "cardholder", "spend controls"

**Use case signals:** Client wants to issue cards to their own users or employees.

---

## Payment Methods

Note which payment methods are relevant to the region:

| Region | Common methods to cover |
| :---- | :---- |
| Europe (SEPA) | Card, SEPA Direct Debit, iDEAL, Bancontact, Klarna |
| UK | Card, BACS Direct Debit, Pay by Bank |
| US | Card, ACH, Link |
| Latin America | Card, PIX (Brazil), OXXO (Mexico) |
| Romania (RO) | Card (primary), local bank transfer |
| Ireland (IE) | Card, SEPA Direct Debit |
| Global | Card (default baseline) |

---

## Embedded Components

**Include this sub-section within Connect if:**
- The platform is building a mobile or web app where connected accounts will manage their account *within* the platform UI (not redirected to Stripe Dashboard)
- "Embedded onboarding", "account management", "embedded payments list", "embedded payouts" mentioned
- `dashboard: none` connected account type

**Components available on mobile (iOS/Android):**
- Account Onboarding (GA)
- Payments (Preview)
- Payouts (Preview)

**Components available on web only:** All other components (reporting, documents, tax, issuing, etc.)
