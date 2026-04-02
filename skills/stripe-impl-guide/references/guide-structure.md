# Guide Structure — Standard Blueprint

Every implementation guide follows this top-level structure. Omit product sections that are not in scope.

---

## Document Header

```markdown
# <ClientName> Stripe Implementation Guide

## Version History

| Date | Author | Version |
| :---- | :---- | :---- |
| <date> | <author> | Initial creation. |
```

---

## 1. Integration Overview

Always present. Covers the engagement context before any technical detail.

### 1.1 Background
1–3 paragraphs. Who the client is, what they are building, why Stripe, and how it fits into their existing architecture. Be specific — avoid generic boilerplate.

### 1.2 Key Goals
Bullet list of the concrete outcomes the integration must achieve. Include a mermaid flowchart showing the high-level architecture (platform, connected accounts, fund flows, apps).

### 1.3 Core Configuration

Configuration table — include whatever applies:

| Property | Value |
| :---- | :---- |
| Account type | Connect Platform / Direct |
| Account ID | acct_XXXXXXXXXX |
| Sandbox Account ID | acct_XXXXXXXXXX |
| Region | [country code] |
| Currencies | [currency list] |
| Connect pricing model | Buy-rate / Blended / N/A |
| Dashboard access for connected accounts | none / express / full |

If environment variables are needed:
```env
STRIPE_SECRET_KEY=sk_live_...
STRIPE_PUBLISHABLE_KEY=pk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
```

### 1.4 API Versions

| API | Version |
| :---- | :---- |
| Stripe API | [version] |
| stripe-java / stripe-node / etc. | [version] |

### 1.5 Implementation Technologies

| Layer | Technology | Role |
| :---- | :---- | :---- |
| Backend | [lang/framework] | Stripe API calls, webhooks |
| [Mobile/Web/etc.] | [tech] | [role] |

### 1.6 SDK Requirements

| SDK | Platform | Purpose |
| :---- | :---- | :---- |
| [sdk name + link] | [platform] | [purpose] |

### 1.7 Embedded Components *(if applicable)*

Only include if Connect embedded components are used. Cover:
- Which components are enabled (onboarding, payments, payouts, etc.)
- Account Session setup and the single-use `acas_...` client secret lifecycle
- Platform-specific setup (Android `EmbeddedComponentManager`, iOS SDK init)

---

## 1a. Project Phases *(optional — include for new-build engagements)*

Only include if the guide is for a new integration with a defined delivery timeline. Omit for guides covering an existing live integration.

Two-column table per month/sprint, covering technical and change-management workstreams in parallel. Example shape:

| Month 1 | Technical | Change Management |
| :---- | :---- | :---- |
| Week 1 | Schema updates, initial Stripe account setup | Activate Stripe account, configure payment methods |
| Week 2 | Core payment flow (API + webhooks) | Configure email branding, payout schedule |
| Week 3 | Saved cards, decline handling | Team training — payout management |
| Week 4 | Automated tests with test cards | Team training — financial reports |

| Month 2 | Technical | Change Management |
| :---- | :---- | :---- |
| Week 5–6 | Bug fixes, functional user tests | Team training — fraud & disputes |
| Week 7 | Polish, confirm account fully configured | Final pre-launch checks |
| Week 8 | **Launch** | **Launch** |

---

## 2. [Product Sections — see product-templates.md]

Insert one section per Stripe product in scope:
- Payments — Checkout Sessions & Payment Intents *(online/card-not-present)*
- Stripe Connect *(platforms, marketplaces, multi-party payments)*
- Stripe Terminal *(in-person, POS, tap-to-pay)*
- Stripe Billing *(subscriptions, invoicing, usage-based)*
- Stripe Radar *(if fraud rules are a focus)*
- Stripe Issuing *(if card issuing is in scope)*

---

## 3. Reporting

Include if reporting is a deliverable or explicitly mentioned in the brief. Use the Reporting template in `references/product-templates.md` for the full sub-section structure. At minimum cover:

- Which reporting mechanisms are in scope (Dashboard, Financial Reports, Reporting API, embedded components)
- Specific reports the client needs — one sub-section each with report type, parameters, data lag, and key columns
- Embedded component availability if the client is surfacing reports to end users within their app

---

## Testing

Always include. Two sub-sections:

### Test Mode
- Stripe testmode vs livemode — separate API keys, separate data
- Key test card numbers for common scenarios:

| Scenario | Card number |
| :---- | :---- |
| Successful payment | `4242 4242 4242 4242` |
| Payment requires 3DS | `4000 0025 0000 3155` |
| Card declined | `4000 0000 0000 9995` |
| Insufficient funds | `4000 0000 0000 9995` |
| Dispute triggered | `4000 0000 0000 2685` |

- Webhook testing: use `stripe listen --forward-to localhost:PORT/webhook` during development
- Identity/address verification test values (for Connect onboarding): refer to Stripe docs test data

### Test Accounts
- Note which Stripe sandbox/testmode account IDs are used
- How to invite team members to the Stripe Dashboard for testing
- Any Stripe-gated features that need to be enabled on the test account

---

## Go-Live Checklist

Always include. Covers the standard Stripe go-live requirements:

- [ ] Switch API keys from test (`sk_test_...`) to live (`sk_live_...`)
- [ ] Webhook endpoint registered in live mode with correct signing secret
- [ ] Idempotency keys used on all state-changing API calls (charges, refunds, etc.)
- [ ] Restricted API keys used where full secret key access is not needed
- [ ] `Stripe-Version` header pinned in all API calls
- [ ] Error handling covers Stripe-specific exceptions (card errors, rate limits, idempotency conflicts)
- [ ] PCI compliance confirmed (SAQ A if using Checkout/Elements; no raw card data on server)
- [ ] AVS / CVC checks reviewed and Radar rules configured
- [ ] Dispute notification webhooks configured and team trained on response process
- [ ] Payout schedule confirmed and bank account verified
- [ ] Stripe Dashboard team access configured with appropriate roles
- [ ] Live payments tested end-to-end with a real card before public launch
- Product-specific additions (add as applicable):
  - [ ] **Connect:** Connected account onboarding tested in live mode with a real business
  - [ ] **Terminal:** Readers registered to live-mode locations; connection token endpoint live
  - [ ] **Billing:** Subscription lifecycle tested (create, upgrade, cancel, failed payment, retry)

---

## Appendix (optional)

- Glossary of client-specific terminology
- Links to Stripe documentation pages referenced in the guide
- Webhook event reference table
- Endpoint reference table (if not already in product sections)
- Resources: Stripe support, docs, API reference, Discord developer community
