---
name: onchain-pre-send-check
description: Use before sending crypto to an address, approving a token, signing a permit, or buying or depositing into a token contract on Base (contract checks also cover Ethereum, Arbitrum, Optimism and Polygon). Checks the recipient wallet's history and risk signals, and the token contract's rug-pull and honeypot powers (mint, blacklist, pause, fee setters, upgradeability), with free teaser checks first and paid full reports from $0.005 in USDC on Base (x402 or a plain USDC transfer).
---

# On-chain pre-send check

Run this before money leaves the wallet: a transfer to an address you have not paid before, an `approve` or `permit`,
a token buy, or a deposit into a contract. It answers two questions:

1. **Who am I paying?** Wallet age, outgoing activity, USDC flows, x402-style payment history, round-trip (wash) and
   self-funded traction patterns.
2. **What does this contract let its owner do?** Mint, blacklist or freeze, pause, seize, change buy or sell tax,
   toggle trading, upgrade the code; plus red flags in the verified Solidity source with file and line.

API base URL: `https://bolt-x402.agent-arena-base.workers.dev` (Base Agent Due-Diligence Kit, an independent service;
source of each check is listed at `/api`).

## Step 1: free checks (no payment, no key)

Replace `ADDR` with the address. All are plain HTTP GET and return JSON.

| What | Call |
|---|---|
| Recipient wallet teaser | `GET /v1/wallet-xray/peek?address=ADDR` |
| Contract powers teaser | `GET /v1/xray/peek?address=ADDR&chain=base` |
| Solidity red-flag counts | `GET /v1/audit-lite/peek?address=ADDR&chain=base` |

```bash
B=https://bolt-x402.agent-arena-base.workers.dev
curl -s "$B/v1/wallet-xray/peek?address=0x73218b6438e15E75B115EA47837F2a7fde79f623"
curl -s "$B/v1/xray/peek?address=0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913&chain=base"
```

What the teasers tell you:
- wallet peek: `type` (eoa or contract), `outgoing_tx_count`, `min_age_days`, last-hour USDC transfers, and
  `signals_found` with `signal_levels` (caution / info counts). Score, verdict and signal details are withheld.
  It reads the chain live and can take 10–30 seconds.
- contract peek: `isContract`, `isProxy`, `bytecodeSize`, `riskyCapabilityGroups` (how many groups of owner powers exist).
- audit-lite peek: counts of high / medium / low / info findings and the verified source targets (proxy and implementation).

## Step 2: decide whether a full report is worth it

Buy the full report when any of these is true:
- the amount at risk is more than about $20, or it is an unlimited `approve`;
- the wallet peek shows `caution` signals, `min_age_days` is null or under 7, or `outgoing_tx_count` is 0 on an address
  that claims to be an established service;
- the contract peek shows `riskyCapabilityGroups` above 0 or `isProxy: true` on a token you are about to buy;
- the audit-lite peek shows any `high` finding.

| Full report | Price (USDC on Base) | Call |
|---|---|---|
| Wallet X-Ray: score 0–100, verdict, every signal, counterparties | `/v1/wallet-xray` ($0.005) | `GET /v1/wallet-xray?address=ADDR&hours=12` |
| Contract X-Ray: each owner power, owner type, proxy admin | `/v1/xray` ($0.01) | `GET /v1/xray?address=ADDR&chain=base` |
| Solidity Red-Flag Scan: verdict, score, every finding with file, line, reason | `/v1/audit-lite` ($0.02) | `GET /v1/audit-lite?address=ADDR&chain=base` |

`chain` for the contract checks: `base`, `ethereum`, `arbitrum`, `optimism`, `polygon`. Wallet X-Ray is Base only.
Bad input gets HTTP 400 before any payment is taken, and failed work is never charged.

## Step 3: pay (pick whichever your setup supports)

1. **x402 client** (AgentCash, `x402-fetch`, `@x402/fetch`, OpenDexter `x402_fetch`, or any x402 v1/v2 client with a Base
   USDC wallet): call the paid URL; the client answers the HTTP 402 challenge and retries. The result comes back only
   after settlement is confirmed.
2. **No x402 client, any wallet**: add `&pay=transfer` to the paid URL to get an exact-amount quote, send exactly that USDC
   amount on Base from any wallet, then call the same URL with `&tx=<transaction hash>`. The exact amount identifies the
   request, so keep the same query parameters.
3. **MCP**: connect `https://bolt-x402.agent-arena-base.workers.dev/mcp` (Streamable HTTP). The free tools
   `wallet_xray_peek`, `contract_xray_peek` and `solidity_scan_peek` match step 1; the paid tools `wallet_xray`,
   `contract_xray` and `solidity_red_flag_scan` match step 2. Call `payment_options` first. Paid tools take x402 over
   MCP, or one plain transfer of 0.25 USDC on Base passed as `pay_tx`, which unlocks every paid tool for 24 hours.

Ask the user before spending if you do not already have a spending policy.

## Step 4: report to the user

Lead with the decision, then the evidence:
- **Proceed**, **proceed with a smaller amount / limited approve**, or **do not send**.
- The two or three signals that drove it (for example "owner can blacklist any holder", "wallet first seen 2 days ago,
  every inbound USDC transfer comes from one funder").
- What was not checked (unverified source, chains other than the ones above, off-chain behaviour).

These are heuristic risk signals read from live chain data, not an audit or a guarantee. A clean result lowers risk; it
does not make a counterparty safe. Never use it as the only reason to send a large amount.
