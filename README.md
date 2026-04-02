# stripe-impl-guide

A Claude Code plugin that generates and maintains Stripe implementation guides for client engagements.

## What it does

Provides a `/stripe-impl-guide` slash command that:

1. Reads a client brief (name, use case, Stripe products)
2. Confirms scope before generating
3. Optionally enriches the guide with live data from the Stripe MCP server (account IDs, products, prices)
4. Generates a structured markdown guide — or updates an existing one

Supports: **Payments** (Checkout Sessions, Payment Intents + Elements), **Connect** (Platform, Buy-rate/Blended, Embedded Components), **Terminal** (POS, Tap-to-Pay, Offline), **Billing** (Subscriptions, Invoicing, Usage-based), **Radar** (Fraud rules, 3DS, Early Fraud Warnings), **Reporting** (Financial Reports, Reporting API, Embedded Components), **Webhooks**

## Installation

Run these two commands in any Claude Code project:

```
/plugin marketplace add pkht/stripe-impl-guide-plugin
/plugin install stripe-impl-guide@stripe-impl-guide
```

## Usage

```
/stripe-impl-guide <client brief>
```

Examples:

```
/stripe-impl-guide "Acme Corp — marketplace connecting freelancers and clients, Connect platform, payouts to freelancers, Node backend"

/stripe-impl-guide "RetailCo — in-person POS with tap-to-pay via Android app, Connect buy-rate, Java Spring Boot backend"

/stripe-impl-guide "SaaS Co — subscription billing, three pricing tiers, usage-based top-up, Stripe Billing, Python backend"
```

The skill will:
- Confirm the detected scope with you before generating
- Create `<ClientName> - Stripe Implementation Guide.md` in the project if none exists
- Update an existing guide if one is found

## Optional: Stripe MCP server

If the Stripe MCP server is configured in your project, the skill will pull live account data (account IDs, products, prices, balance) directly into the guide.

Add to `.mcp.json` in your project:

```json
{
  "stripe": {
    "type": "http",
    "url": "https://mcp.stripe.com",
    "headers": {
      "Authorization": "Bearer ${STRIPE_API_KEY}"
    }
  }
}
```

Without it, the skill uses clearly marked placeholders that you fill in manually.

## File structure

```
stripe-impl-guide-plugin/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── README.md
└── skills/
    └── stripe-impl-guide/
        ├── SKILL.md                      # Slash command definition
        └── references/
            ├── guide-structure.md        # Standard document blueprint
            ├── product-detection.md      # Identifying Stripe products from a brief
            └── product-templates.md      # Sub-section templates per product
```
