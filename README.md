# stripe-impl-guide

A Claude Code plugin that generates and maintains Stripe implementation guides for client engagements.

## What it does

Provides a `/stripe-impl-guide` slash command that:

1. Reads a client brief (name, use case, Stripe products)
2. Searches the current project for an existing implementation guide
3. Optionally enriches the guide with live data from the Stripe MCP server (account IDs, products, prices)
4. Generates a structured markdown guide — or updates an existing one

Supports: **Payments** (Checkout Sessions, Payment Intents + Elements), **Connect** (Platform, Buy-rate/Blended, Embedded Components), **Terminal** (POS, Tap-to-Pay, Offline), **Billing** (Subscriptions, Invoicing, Usage-based), **Radar** (Fraud rules, 3DS, Early Fraud Warnings), **Reporting** (Financial Reports, Reporting API, Embedded Components), **Webhooks**

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
- Confirm scope with you before generating
- Detect which Stripe products are needed from the brief
- Create `<ClientName> - Stripe Implementation Guide.md` in the project if it doesn't exist
- Update an existing guide if one is found

## Dependencies

### Optional: Stripe MCP server

If the [Stripe MCP server](https://github.com/stripe/agent-toolkit) is configured in your project, the skill will use it to pull live account data (account IDs, products, prices, balance) into the guide automatically.

To configure the Stripe MCP server, add to `.mcp.json` in your project:

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

Without the MCP server the skill works fine — it uses placeholder values that you fill in manually.

## Installation

### As a local plugin (development)

```bash
# From your Claude project root
claude plugin install /path/to/stripe-impl-guide-plugin
```

### From GitHub (once published)

```bash
claude plugin install github:<your-org>/stripe-impl-guide
```

## File structure

```
stripe-impl-guide-plugin/
├── .claude-plugin/
│   └── plugin.json
├── README.md
└── skills/
    └── stripe-impl-guide/
        ├── SKILL.md                          # Slash command definition
        └── references/
            ├── guide-structure.md            # Standard document blueprint
            ├── product-detection.md          # Identifying Stripe products from a brief
            └── product-templates.md          # Sub-section templates per product
```
