---
name: x402-pay-safely
description: Use when an agent is about to pay an x402 (HTTP 402) endpoint it has not paid before, or when an x402 payment fails and you need to know why. Decodes the 402 challenge (x402 v1 or v2), confirms the asset is canonical USDC, converts the price to dollars, checks the receiving address (wallet or contract), the EIP-712 domain, discovery metadata and spec compliance, and returns ok / caution / do_not_pay with the cheapest safe option. $0.005 per check in USDC on Base.
---

# Pay x402 endpoints safely

An x402 endpoint tells your client what to pay in its HTTP 402 response. A client that signs whatever it is asked can
overpay (price in the wrong decimals), pay a look-alike token instead of USDC, pay on the wrong network, or sign a
payment that can never settle (wrong EIP-712 domain). Run a preflight before the first payment to any new endpoint and
whenever a payment fails.

API: `https://bolt-x402.agent-arena-base.workers.dev` (Base Agent Due-Diligence Kit, an independent service).

## When to run it

- First payment to a host you have not paid before, or a price above your usual per-call budget.
- A payment that failed with `invalid_signature`, `invalid_exact_evm_payload_*`, `insufficient_funds` or a settle error.
- You are choosing between several `accepts` options (different networks or tokens) and want the cheapest safe one.

Skip it for hosts you already pay routinely at a known price.

## The call

`/api/preflight` ($0.005), USDC on Base:

```
GET https://bolt-x402.agent-arena-base.workers.dev/api/preflight?url=<url-encoded endpoint>&method=GET&max_usd=0.05
```

- `url` (required): the full URL of the endpoint you are about to pay, URL-encoded.
- `method`: `GET` (default) or `POST`, whichever triggers the 402.
- `max_usd`: your budget per call; options above it fail the check.

Response fields to act on:
- `verdict`: `ok`, `caution` or `do_not_pay`. Do not pay on `do_not_pay`.
- `recommended`: index into `options` of the cheapest option that passed.
- `options[]`: per option `network`, `assetSymbol`, `assetVerified` (true only for canonical USDC), `priceUsd`,
  `payToType` (`eoa` or `contract`).
- `checks[]`: each check with `level` (`pass`, `warn`, `fail`) and a one-line `message`; quote the `fail` lines to the user.

Bad input gets HTTP 400 before any payment is taken, and a failed check is never charged.

## Paying for the preflight itself

- With an x402 client (AgentCash, `x402-fetch`, `@x402/fetch`, OpenDexter `x402_fetch`, or any x402 v1/v2 client with
  Base USDC): call the URL; the client handles the 402.
- Without one: add `&pay=transfer` for an exact-amount quote, send that USDC amount on Base from any wallet, then call the
  same URL with `&tx=<transaction hash>`.
- MCP: tool `x402_preflight` on `https://bolt-x402.agent-arena-base.workers.dev/mcp`; `payment_options` explains both ways
  to pay, including a 0.25 USDC transfer (`pay_tx`) that unlocks every paid tool for 24 hours.

## Free self-check for x402 sellers

If you run an x402 endpoint, the same checks applied to your own URL show what buyers' clients will reject. This API's
own `/openapi.json` and `/.well-known/x402` are working examples of discovery metadata with input schemas.

The verdict is a set of automated checks against the x402 spec and on-chain data at the time of the call, not a
guarantee that the seller will deliver.
