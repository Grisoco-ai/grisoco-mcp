# Grisoco AI — security audits for AI agents (MCP)

Let your AI agent order its own smart-contract security audit. Grisoco AI audits **Solidity on any EVM chain** and **DAML (Canton)** packages, and exposes the whole flow as a remote [MCP](https://modelcontextprotocol.io) server: your agent quotes, submits, polls and collects the report on its own, paid from your prepaid Grisoco balance.

- **MCP endpoint:** `https://grisoco.com/mcp` (Streamable HTTP)
- **Auth:** a Grisoco API key, sent as `Authorization: Bearer grisoco_sk_...`
- **Docs:** https://grisoco.com/developers
- **Registry:** `com.grisoco/audits` on the [official MCP Registry](https://registry.modelcontextprotocol.io)

This repository holds the listing and setup instructions. The server itself is hosted by Grisoco; there is nothing to install or run.

## What it audits

- **Solidity on EVM chains:** Ethereum, Polygon, BNB Chain, Arbitrum, Optimism, Base.
- **DAML packages for Canton (Daml 3.x):** authorization, privacy, contention and liveness, interfaces and upgrades. Authorization findings are proven by execution.
- **AI-agent payment accounts:** smart accounts and session keys (ERC-4337, ERC-7579, ERC-7710) get a dedicated delegation review covering spend limits, permission storage, delegation narrowing, expiry and revocation, and the reach of each granted scope.

Each audit runs deterministic tools and a deep AI review, checks every finding before it is reported, and returns one report with severity, location, a proof-of-concept plan and a remedy per finding.

## Get a key

1. Sign in at [grisoco.com](https://grisoco.com/app) and top up your balance with USDC.
2. Open **Create API key**, create a key and set a **daily spending cap**: the most that key may spend per day, whatever your balance.
3. Copy the key (it starts with `grisoco_sk_`). It is shown once; keep it in a secret store, never in source code.

## Connect

**Claude Code**

```bash
claude mcp add --transport http grisoco https://grisoco.com/mcp \
  --header "Authorization: Bearer grisoco_sk_..."
```

**Any client that reads an `mcpServers` config** (Claude Desktop, Cursor, Windsurf and others)

```json
{
  "mcpServers": {
    "grisoco": {
      "type": "http",
      "url": "https://grisoco.com/mcp",
      "headers": { "Authorization": "Bearer grisoco_sk_..." }
    }
  }
}
```

## Tools

| Tool | What it does |
|---|---|
| `quote_audit` | Prices an audit without charging anything. |
| `submit_audit` | Starts an audit. Requires `max_price_cents`, so the agent always states what it agreed to pay; refused if the price is higher. |
| `get_audit` | Status: queued, running, succeeded, failed (refunded) or expired. Poll about every 30 seconds. |
| `get_audit_report` | The finished report as Markdown or JSON. |
| `get_balance` | The account's prepaid balance, in US cents. |
| `list_audits` | The account's most recent audits. |

A typical flow: `quote_audit` → `submit_audit` with the quoted price → poll `get_audit` → `get_audit_report`.

## Pricing and limits

- Priced per contract, no subscription. Files that are only imported are free. Current prices: https://grisoco.com/pricing
- An audit that fails on our side is refunded automatically.
- Each key has a daily spending cap, and keys can only run and read audits, run transaction checks and see the balance.

## REST

The same flow is available over plain REST (`POST /v1/audits/quote`, `POST /v1/audits`, `GET /v1/audits/{id}`, `GET /v1/audits/{id}/report`). See https://grisoco.com/developers.

## Also with the same key: Agent Transaction Check

Before your agent signs an **on-chain payment** (a token sent to one recipient on Base, Ethereum or Polygon, as a transaction or a gasless EIP-3009 / x402 authorization), it can ask Grisoco whether the transaction does exactly what it means to pay: the right token, the exact amount, the right recipient, and nothing else. The answer is `allow`, `warn` or `block`, with a reason per check. Rule-based, no AI.

- **Endpoint:** `POST https://check.grisoco.com/v1/check` (REST; not an MCP tool yet)
- **Payments only:** swaps, deposits, mints and other calls are not checked
- **Two rules:** take the intent (token, amount, recipient) from your user's task, never from the payment request; and fail closed — sign only on `allow`, never on an error or timeout
- **Price:** per verdict, from the same balance: https://grisoco.com/pricing
- **Guide:** https://grisoco.com/transaction-check · **OpenAPI:** https://grisoco.com/transaction-check/openapi.json

## Important

Grisoco AI is an automated audit, not a certification: no human security engineer reviews the code, and no audit can prove a contract is free of vulnerabilities. Have a qualified engineer review every finding before relying on a contract that holds real value. See the [Terms](https://grisoco.com/terms) and [Privacy Policy](https://grisoco.com/privacy).

Contact: contact@grisoco.com
