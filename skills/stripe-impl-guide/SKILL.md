---
name: stripe-impl-guide
description: Generate or update a Stripe implementation guide for a client engagement. Use when the user invokes /stripe-impl-guide with a client name or brief.
argument-hint: "<client brief — name, use case, Stripe products>"
allowed-tools: [Read, Write, Edit, Glob, Grep, mcp__stripe__get_stripe_account_info, mcp__stripe__fetch_stripe_resources, mcp__stripe__search_stripe_documentation, mcp__stripe__list_products, mcp__stripe__list_prices, mcp__stripe__retrieve_balance, mcp__stripe__list_customers, mcp__stripe__list_subscriptions]
---

# Stripe Implementation Guide Skill

You are generating or updating a professional Stripe implementation guide for a client engagement.

The user has invoked this skill with: `$ARGUMENTS`

All structure, templates, and product knowledge you need are in the `references/` directory alongside this file. Do not rely on finding example guides or template files in the host project — treat every project as a blank slate.

---

## Phase 1 — Gather Context

### 1a. Parse the brief

Extract from `$ARGUMENTS`:
- **Client name** — the company the guide is for
- **Stripe products in scope** — look for explicit mentions (Connect, Terminal, Billing, Radar, Issuing, etc.) or infer from the use case description (see `references/product-detection.md`)
- **Tech stack** — backend language, mobile platform, frontend framework if mentioned
- **Region / country** — used for currency, payment methods, and compliance notes
- **Account type** — platform with connected accounts? Direct? Buy-rate vs blended pricing?

### 1b. Check for an existing guide to update

Search the project for a guide that may already exist for this client:

```
Glob: **/*<ClientName>*.md
Glob: **/*Stripe*.md
Grep: "Stripe Implementation" in *.md
```

If a guide is found, read it. Your job becomes updating it — add missing sections, fill in placeholders, update configuration tables. Do not regenerate content that is already accurate.

If nothing is found, you are creating from scratch.

### 1c. Attempt Stripe MCP enrichment

Try the following MCP tools if the Stripe MCP server is available. These are optional — skip gracefully if unavailable or no account is configured:

- `mcp__stripe__get_stripe_account_info` — platform account ID, country, capabilities
- `mcp__stripe__list_products` — configured products/plans (Billing guides)
- `mcp__stripe__list_prices` — pricing tiers (Billing guides)
- `mcp__stripe__list_customers` — customer data shape
- `mcp__stripe__list_subscriptions` — confirm subscription model in use
- `mcp__stripe__retrieve_balance` — platform balance structure (Connect)

Incorporate any live values into the guide's configuration tables. Where live data is unavailable, use clearly marked placeholders: `acct_XXXXXXXXXX`, `prod_XXXXXXXXXX`, `price_XXXXXXXXXX`, etc.

---

## Phase 2 — Determine Scope

Using the brief and any MCP data, confirm which Stripe products are in scope. Refer to `references/product-detection.md` for signals and heuristics.

Output a scope summary before generating content:

```
Client: [name]
Products: [Connect | Terminal | Billing | Direct Payments | Radar | ...]
Connect type: [Platform / Direct] — [Buy-rate / Blended / N/A]
Backend: [language/framework or unknown]
Mobile/Frontend: [iOS / Android / React Native / Web / N/A]
Region: [country code or unknown]
Existing guide: [yes — path | no]
MCP data: [yes | partial | no]
```

Ask the user to confirm or correct this scope before proceeding to generate the full guide. This avoids generating a large document in the wrong direction.

---

## Phase 3 — Generate the Guide

Read `references/guide-structure.md` for the document blueprint. Read `references/product-templates.md` for the sub-section structure of each product area. Include only sections that are in scope — do not add placeholder sections for products not in the brief.

### Writing conventions

- Use **mermaid** for flow diagrams (fund flows, state machines, sequence diagrams). Keep them simple and accurate.
- Use **tables** for configuration properties, SDK requirements, endpoint references, and webhook event lists.
- Use **fenced code blocks** with the correct language tag. Default backend language is Java unless the brief specifies otherwise. Mobile: Kotlin for Android, Swift for iOS.
- Mark genuinely unknown values with `> **TODO:** [what is needed]` — don't invent values.
- Use `> **Note:**` for important caveats (security, ordering constraints, single-use tokens, etc.).
- Keep prose tight. Tables and code carry the detail — narrative text should only explain what isn't self-evident.

### Code quality bar

- Show real Stripe SDK calls, not pseudocode. Use the SDK for the backend language in the brief.
- Include error handling for Stripe-specific exceptions (`StripeException`, `StripeError`, etc.).
- Use realistic field names and types.
- Never include real secret keys, account IDs, or customer data. Use `.env` placeholder format for keys.

---

## Phase 4 — Write the Output

### Updating an existing guide
Use `Edit` to add or update sections in place. Do not overwrite accurate content. Add a version history row if the guide has a version table.

### Creating a new guide
Write the file to the project root (or a subdirectory the user specifies):

```
<ClientName> - Stripe Implementation Guide.md
```

Do not assume an `implementation_guide/` folder exists — write to wherever makes sense in the project, or ask the user if unsure.

---

## Reference Materials

All knowledge for generating guides lives here — no external files required:

- `references/guide-structure.md` — document blueprint: section order, Project Phases, Testing, Go-Live Checklist
- `references/product-detection.md` — how to identify Stripe products from a client brief
- `references/product-templates.md` — sub-section templates for Connect, Terminal, Billing, Direct Payments, Radar, Webhooks
