# Hermes model-driven Nano payout assay

Status: **closed 2026-09-14 — bought, paid, delivered, verified, and the Ӿ3
report fee received and confirmed on chain.**

This assay tested a narrow autonomy claim: can a scheduled Hermes agent—not a
human operator choosing at transaction time—decide whether to buy a real service
with Nano, execute the payment from a locally held seed, verify delivery, and
leave enough public evidence for the decision to be audited?

**Answer: yes, once, under the boundaries below.** The full transcript,
evidence, and limits are in the [run record](#run-record--2026-09-14-utc).

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

## Funding — 2026-09-14

Buyer confirmed the hold for wanted item 2(b) (Hermes runtime) and seeded
0.05 XNO (block `B7D8504E0D42D15B21011F520ACA954DFF1E97D03AABE56489CA70820D2C7CB3`,
ledger #68 on pursekeeper.dev/log). The report is therefore labelled seeded and
incentivized, like the two fills before it.

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

## Run record — 2026-09-14 UTC

Runtime: scheduled Hermes cron session (unattended; operator not present),
Hermes Agent 0.20.1, model `sprintcx/tier-1` through the configured custom
provider.
Model-visible decision sequence, in order:

1. **Wakeup and state read.** `kolonie.wakeup`, ticket/board state. GitHub
   notification surfaced the buyer's hold confirmation on pursekeeper/api#5
   (2026-09-14T08:08Z).
2. **Funding verified.** `GET /v1/account_info` → unopened, balance 0.
   `GET /v1/receivable` → exactly one confirmed send of 0.05 XNO from
   `nano_1xug1q5t7nxoj3ywwzokiea9jz8fq8qfgzp8pbyfr3co3e5xgj755uofu8ue`,
   matching the buyer's stated seed block.
3. **Seller catalogue inspected live.** `GET https://pursekeeper.dev/sellers.json`
   (2026-09-14T19:22:03Z). Reachable-and-verified sellers: nanogpt (Ӿ0.00108,
   4588 ms), pyfile-llm (Ӿ0.001, 104 ms), contract-lens (Ӿ0.01), feed-weight-check
   (Ӿ0.01), oreomuncher-attest (USD-pegged, 0.0059–0.0149 XNO, 195 ms).
   Considered and rejected: cleartable, llmrt, stringsafeqa (unreachable);
   nanogpt/pyfile-llm (a completion would be produced but adds no verifiable
   artifact to the audit); contract-lens/feed-weight-check (viable alternates).
4. **Merchant chosen: oreomuncher-attest** (`https://api.shehriyar.ink/v1/attest/response`).
   Reason recorded before payment: the attestation is a real third-party
   service whose output strengthens the audit evidence of this paid assay
   itself; the endpoint is reachable and previously verified (ledger #75); the
   live price is within the 0.02 XNO cap.
5. **Receive.** Pocketed the 0.05 XNO send with a signed open block:
   `4C86FEE980C44A9574799496700887DDEE4F13EE7FC964100A52F3C3643598C0`.
6. **Quote inspected before paying.** Unpaid POST answered HTTP 402 with an
   x402 v2 `PAYMENT-REQUIRED` challenge: nano:mainnet exact rail,
   0.015071 XNO to `nano_1zqdw3qf1z8k3jx8jintaiwpo3yz7zqh1me4ph5j439ts8hsppx8dzy4xcsz`
   (USDC-on-Base rail quoted alongside). Cap check passed (0.015071 ≤ 0.02).
7. **Paid call.** Signed send block
   `3080E11AFBAABCB8F3500143C628AE0C7D79E06DF0C601E8907F874D3F465051`
   (work via pursekeeper `/v1/work`), retried with `PAYMENT-SIGNATURE`.
   HTTP 200 in the same second; settlement header named the same block hash.
8. **Delivery verified independently.**
   - Nano: send confirmed by `pursekeeper.dev/v1/verify`, `rpc.nano.to`, and
     `node.somenano.com` (subtype send, amount 0.015071, height 2).
   - Attestation: Ed25519 signature over the canonical attestation document
     verified **offline** with Node `crypto` against the published key
     (`pHgeKhYwf908uO1fkknXp2cckVpQ1O+lzvRPyeDrbTQ=`); positive verification
     true, mutated-label negative control false.
9. **Balance after.** 0.034929 XNO (open 0.05 − send 0.015071). Seed remains
   local, mode 0600, never published.

### Paid-call artifacts

- Raw request body (sha256 `2b9bc9ba02e18aa2c5848a897b3e802486d45e15029c126ff687db06ead70d9c`):
  `{"payload":{...decision record...},"label":"assay-2b-hermes-2026-09-14","include_anchor":true}`
- 402 terms: nano:mainnet exact, `15071000000000000000000000000` raw, payTo
  `nano_1zqdw3qf1z8k3jx8jintaiwpo3yz7zqh1me4ph5j439ts8hsppx8dzy4xcsz`,
  maxTimeoutSeconds 300, work required at `fffffff800000000`.
- Send block: `3080E11AFBAABCB8F3500143C628AE0C7D79E06DF0C601E8907F874D3F465051`
- Delivered attestation: canonical_sha256
  `f95bffbef2e6ddded4ab8a41f3aa23a57c4de2019b56c95fadb5989c1ba290b0`,
  raw_sha256 `46c44e2419c22ca6e37d9eaeb95578d568e43b610d3abae939f9351356bc8c26`,
  issued_at `2026-09-14T19:26:45.318697Z`, Ed25519 signature
  `Zy3ndhH9FSmKMPjVEzNaWKn2D1+HG7beIeC9virTxBIM18yBCzaXgT4bBhumB3EDAcBe/fwFWxuulHVQBgr1CA==`,
  Base anchor block 51312329.
- Versions: Hermes Agent 0.20.1, Node v26.7.0, `nanocurrency` 2.5.0,
  `no-node.js` (SHA-256 `43c82feb…`) unmodified; paid-call script derived from
  `client-x402.js` and kept local with the seed path.

Public, secret-free machine evidence and the exact model-visible decision
statements are in [`evidence.json`](evidence.json) and
[`transcript.md`](transcript.md).

## Limits

- One host, one decision, one merchant, one payment.
- No claim that a human could not have produced the same transaction manually;
  the claim tested is that the model chose within a published contract in an
  unattended scheduled run, with the seed never leaving the host.
- The stated reason for buying includes strengthening this assay's own
  evidence, and the purchase was made from buyer-seeded funds; both facts are
  part of the record, not hidden.
- The Ӿ3 report fee from pursekeeper is the incentivized part; it was paid on
  delivery acceptance and is recorded under [Fee](#fee) below.

## Fee

Buyer pursekeeper promised Ӿ3 on delivery for wanted item 2(b) (Hermes) as a
second Hermes report after the Daltonray625 first fill. Delivery:
https://github.com/pursekeeper/api/issues/5 (report comment).

- Fee send block: `8B07595CA707D14C00772A10D3ADB761423021EC5ED18C82699AE64B6A0EE039`
  (from `nano_1xug1q5t7nxoj3ywwzokiea9jz8fq8qfgzp8pbyfr3co3e5xgj755uofu8ue`, amount 3 XNO, confirmed).
- Pocket receive block: `23D8526FD0610B7275EF54690829D8742130070614027165FB1F64B53A168C1A`
  (account `nano_15s4fd48jztkijifk1agr5gtnkos55gex8nbzom7qxrt3hf985pbkjik3s84`, height 3, balance 3.034929 XNO).
- Verification: confirmed by pursekeeper `/v1/verify`, `rpc.nano.to`, and `node.somenano.com/proxy`.
- Remaining wallet balance: 3.034929 XNO.
