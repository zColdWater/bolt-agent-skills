---
name: defi-deposit-check
description: Use before depositing into, swapping through, or voting on a DeFi protocol on Base (currently Aero / Aerodrome / Velodrome, Morpho Blue and Moonwell). Answers who can upgrade, pause or redirect user funds - admin keys, multisig or emergency council, timelock delays, governance attack cost - what admins cannot do, past incidents and declared unknowns, every claim sourced. Free catalog, previews and governance-attack-cost calculator; full dossier $0.01 in USDC on Base.
---

# DeFi deposit check (control and governance)

Before putting funds into a protocol, find out who can change the rules on you and how quickly. Smart-contract audits
answer "is the code buggy"; this answers "who holds the keys, what can they do with them, and what does it cost to
capture governance".

API: `https://bolt-x402.agent-arena-base.workers.dev` (Base Agent Due-Diligence Kit; dossiers are AI-written research,
every claim carries a source and a confidence tag).

## Step 1: is the protocol covered? (free)

```
GET https://bolt-x402.agent-arena-base.workers.dev/api/sage
```

Returns each dossier's `slug`, `name`, `as_of` date, `headline`, `attention` level, scores (upgradeability, admin power,
governance capture, lifecycle, incidents; null = not scored yet) and the count of declared unknowns and sources.
Current slugs: `aero-base`, `aerodrome-base` (legacy, superseded by Aero), `moonwell-base`, `morpho-blue-base`.

If the protocol is not in the catalog, say so and fall back to a contract check (see the `onchain-pre-send-check`
skill) on the exact contract you would deposit into.

## Step 2: free preview and governance attack cost

- `GET /api/sage/preview?slug=moonwell-base` - headline findings without the full evidence.
- `GET /api/sage/gac?slug=moonwell-base&price=0.01&var=50000000` - Governance Attack Cost: a lower bound on what it
  would cost to buy enough votes to reach quorum, from the live governance-token `price` in USD (supply your own) and the
  value at risk `var` in USD. It returns `gacToValueAtRiskPct` and a band (green / amber / red); an attack that is cheap
  relative to the funds governance controls is a red flag. Also `quorum=` instead of `slug=` for protocols not covered.

## Step 3: full dossier when real money is involved

`/api/sage/dossier` ($0.01), USDC on Base. Pass exactly one of:
- `slug` - one of the slugs above;
- `protocol` - a name such as `aero`, `aerodrome`, `velodrome`, `morpho`, `moonwell`;
- `address` - a contract you are about to use (for example the Morpho Blue core contract, the AERO token, VotingEscrow).
  An address that is not covered gets HTTP 400 and is not charged.

```
GET https://bolt-x402.agent-arena-base.workers.dev/api/sage/dossier?protocol=morpho
```

For background reading, `/api/sage/report` ($0.10) is a long-form markdown report, "Who Can Move Your Money?", covering
Morpho Blue, Aerodrome and Moonwell, the Governance Attack Cost framework and a pre-deposit checklist.

## How to pay

- x402 client (AgentCash, `x402-fetch`, `@x402/fetch`, OpenDexter `x402_fetch`, or any x402 v1/v2 client with Base USDC):
  call the URL and let the client answer the 402.
- No x402 client: add `&pay=transfer` for an exact-amount quote, send that USDC amount on Base from any wallet, then call
  the same URL with `&tx=<transaction hash>`.
- MCP: tools `defi_dossier_catalog`, `defi_dossier_preview`, `governance_attack_cost` (free) and `defi_control_dossier`,
  `defi_research_report` (paid) on `https://bolt-x402.agent-arena-base.workers.dev/mcp`; `payment_options` explains a
  0.25 USDC transfer (`pay_tx`) that unlocks every paid tool for 24 hours.

## Report to the user

State who can move or freeze funds and how fast (timelock or none), whether the key holders are published, the
governance attack cost against the value at stake, and the declared unknowns. Keep the dossier's `as_of` date in the
answer: control setups change. This is research, not investment advice and not an audit.
