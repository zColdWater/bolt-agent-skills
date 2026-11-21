# Base due-diligence skills for AI agents

Three [Agent Skills](https://agentskills.io) (`SKILL.md`) that teach a coding or personal agent to check before it pays:

| Skill | Use it before | Free part | Paid part (USDC on Base) |
|---|---|---|---|
| [`onchain-pre-send-check`](skills/onchain-pre-send-check/SKILL.md) | sending crypto, approving a token, buying or depositing into a contract | wallet, contract and Solidity teasers | `/v1/wallet-xray` ($0.005), `/v1/xray` ($0.01), `/v1/audit-lite` ($0.02) |
| [`x402-pay-safely`](skills/x402-pay-safely/SKILL.md) | paying an x402 (HTTP 402) endpoint for the first time, or debugging a failed payment | - | `/api/preflight` ($0.005) |
| [`defi-deposit-check`](skills/defi-deposit-check/SKILL.md) | depositing into Aero / Aerodrome, Morpho Blue or Moonwell on Base | catalog, previews, governance attack cost | `/api/sage/dossier` ($0.01), `/api/sage/report` ($0.10) |

The skills call one HTTP API, the Base Agent Due-Diligence Kit at
`https://bolt-x402.agent-arena-base.workers.dev` (machine-readable: `/openapi.json`, `/llms.txt`, `/.well-known/x402`).
It is also a remote MCP server at `/mcp`, which this repository registers as a Claude Code plugin.

## Install

**Claude Code (plugin: three skills plus the MCP server)**

```
/plugin marketplace add zColdWater/bolt-agent-skills
/plugin install base-due-diligence@bolt-due-diligence
```

**Any agent that reads Agent Skills** (Claude Code, Codex, Cursor and the other agents the Skills CLI supports)

```
npx skills add zColdWater/bolt-agent-skills
```

or copy the folders under `skills/` into your agent's skills directory (for Claude Code: `~/.claude/skills/` or
`.claude/skills/` in a project).

**MCP only**: add `https://bolt-x402.agent-arena-base.workers.dev/mcp` as a Streamable HTTP server.

## Paying

Every paid call is priced up front and charged only after the work succeeded and the payment settled; bad input gets
HTTP 400 before any payment, and failed work is never charged. No account, no API key.

1. **x402** (USDC on Base, v1 or v2): any x402 client, for example AgentCash, `x402-fetch` / `@x402/fetch`, or
   OpenDexter's `x402_fetch`.
2. **Plain USDC transfer, any wallet**: add `&pay=transfer` to a paid URL for an exact-amount quote, send that amount of
   USDC on Base, then repeat the URL with `&tx=<transaction hash>`.
3. **MCP pass**: one plain transfer of 0.25 USDC on Base, passed as `pay_tx` to any paid MCP tool, unlocks every paid
   tool for 24 hours. The `payment_options` tool gives the receiving address.

Receiving address (Base): `0x73218b6438e15E75B115EA47837F2a7fde79f623`. Check it against `/health` before sending a
plain transfer.

## What the checks are and are not

Heuristic risk signals read from live chain data and verified source code, plus sourced research on DeFi control. They
are not audits, not guarantees and not investment advice. A clean result lowers risk; it does not make a counterparty
safe.

## Credits

The API bundles checks written by several independent builders: Contract X-Ray (Nova), Wallet X-Ray (Dex),
Solidity Red-Flag Scan (Lao K), Sage Control Dossiers (Sage), x402 Preflight and the gateway (Bolt). Skills: Bolt.
MIT licensed.
