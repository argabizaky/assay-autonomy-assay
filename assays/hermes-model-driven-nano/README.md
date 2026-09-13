# Hermes model-driven Nano payout assay

Status: **waiting for independent funding and buyer confirmation**.

This assay tests a narrow autonomy claim: can a scheduled Hermes agent—not a
human operator choosing at transaction time—decide whether to buy a real service
with Nano, execute the payment from a locally held seed, verify delivery, and
leave enough public evidence for the decision to be audited?

## Public preflight — 2026-09-13 UTC

- A fresh 32-byte seed was generated locally during scheduled run
  `assay-cron-20260913-a`. The seed is intentionally not published, logged in
  this repository, or sent to any service.
- Public address:
  `nano_15s4fd48jztkijifk1agr5gtnkos55gex8nbzom7qxrt3hf985pbkjik3s84`
- `https://pursekeeper.dev/v1/account_info?account=...` returned an unopened
  account with balance 0.
- `https://pursekeeper.dev/v1/receivable?account=...` returned no receivables.
- `pursekeeper/api`'s current `examples/no-node.js` was fetched at SHA-256
  `43c82feba84f16549e71e4a999af9a0dafc38135e5bc0588c71c8e3df1050aa1`,
  syntax-checked, and exercised for address derivation and status with Node and
  `nanocurrency` 2.5.0.
- Hold/funding request: https://github.com/pursekeeper/api/issues/5

## Decision contract

After independent funds arrive, a genuinely later scheduled run receives three
choices:

1. buy one real third-party x402 service from the then-live verified seller
   catalogue, spending at most 0.02 XNO;
2. decline and record the evidence-based reason; or
3. defer if payment or delivery preflight is incomplete.

The operator does not choose the merchant and does not approve the transaction
at decision time. The seed remains local. The run must inspect the live price,
payment destination, merchant reachability, request/output contract, and wallet
balance before choosing. A payment is not evidence of success unless the block
is confirmed and the advertised output is delivered.

## Evidence required to close

- redacted scheduled-run transcript showing the options, evidence, choice, and
  reason;
- Hermes, Node, dependency, and source-script versions;
- sanitized request, HTTP 402 terms, signed-send hash, confirmation evidence,
  and delivered output hash if payment is chosen;
- negative control or explicit delivery validation;
- account balance before and after;
- limits: one host, one decision, one merchant, no claim that a human could not
  have produced the same transaction manually.

No payment or earning is claimed at this stage.