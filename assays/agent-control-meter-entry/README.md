# Agent Meter fresh-agent entry assay

Status: **measured; no payment made**.

This assay tested the newly published Agent Meter entry path from a clean external Linux host on 2026-09-13 UTC. The narrow question was whether an agent can discover the service, use the advertised five free looks without an account or API key, and reach a bounded machine-readable payment boundary.

## Scope and method

- Read `https://agent-control.net/llms.txt` and fetched the public MCP descriptor at `POST /api/v1/mcp`/`GET /api/v1/mcp` as exposed live.
- Fetched `GET /api/v1/meter/pricing` and `https://agent-control.net/openapi-meter.json`.
- Chose a fresh random `X-Agent-Pass` identifier; no email, signup, API key, wallet key, or seller-supplied credential was used.
- Called `POST /api/v1/meter/scan` five times across Solana, Ethereum, and Base addresses, then a sixth time with the same identifier.
- Used a second fresh identifier for malformed-address and over-cap negative controls.
- Requested the default pass with `POST /api/v1/meter/pass {}` but did not pay it.

## Measured results

### The advertised free-to-paid boundary works

1. Looks 1–5 each returned HTTP 200 and decremented `free_looks_remaining` from 4 to 0.
2. Look 6 returned HTTP 402 with:
   - `sku: look`
   - `price_usd: 0.02`
   - `chain: solana`
   - `asset: usdc`
   - `amount_base_units: "20000"`
   - a unique `invoice_id`, `reference`, locked `pay_to`, and Solana Pay URL.
3. `POST /api/v1/meter/pass {}` independently returned the same priced boundary with a different invoice/reference.
4. The public pricing document agrees on the default `look` price and payee.
5. No human-only step appeared before payment. A wallet-funded agent could proceed by paying the invoice reference and polling `/api/v1/meter/watch`; that paid path was intentionally not exercised in this unpaid entry assay.

### Cap enforcement works

A free `POST /api/v1/meter/preflight` with `value_usd: 1` and `cap_usd: 0.5` returned HTTP 200 with:

```json
{
  "decision": "stop",
  "must_abort": true,
  "reason": "over_cap value_usd exceeds cap_usd."
}
```

This is an advisory control: the public docs correctly state that Agent Meter never holds funds and cannot stop a wallet that ignores the decision.

## Primary defect

`POST /api/v1/meter/scan` does not validate address syntax before returning a risk verdict.

Two deliberately malformed inputs returned HTTP 200 and `risk: "new"`:

- Solana: `not-an-address`
- Ethereum: `0xdeadbeef`

Both responses said `no_history No transfer history on file for this address.` An unsupported chain correctly returned HTTP 400, so chain validation exists while address validation does not.

This matters because the product answers **“Can I pay this address?”**. `new` is a plausible destination-risk category, not an invalid-input error. An agent can therefore interpret an impossible destination as a real but unseen payee and continue to signing, wasting a metered look and moving failure downstream to the wallet.

### Recommended repair

Validate destination syntax for each accepted chain before consuming a look or producing a risk verdict:

- Solana: Base58-decode to exactly 32 bytes and reject invalid/non-canonical strings.
- Ethereum/Base: require exactly 20 bytes, with optional checksum validation when mixed case is supplied.
- Return HTTP 400 with a stable machine-readable code such as `invalid_address` and do **not** decrement the free/paid look allowance.
- Add malformed, short, wrong-length, and valid-zero-address fixtures per chain.
- Apply the same validation to `preflight` and `scan-batch` so the answer surface is consistent.

## Documentation drift

The live pricing catalogue includes `pass_1h` at $0.25 as `catalog-only`, while the OpenAPI request enum for `POST /api/v1/meter/pass` contains only `look`, `looks_20`, `addresses_100`, and `stamp_tx`. That exclusion is consistent with the current `llms.txt` wording, but OpenAPI exposes only three of the nine public Meter endpoints named by pricing/MCP. Discovery therefore works best through `llms.txt` or MCP, not through OpenAPI alone.

## Limits

- One external host and two fresh anonymous identifiers.
- Five free scans, one free preflight, and two unpaid 402 observations.
- No USDC was sent; `/watch`, paid-pass issuance, post-payment scan, batch, and stamp were not tested.
- The sampled valid-looking addresses all returned `new`; this does not test the quality or provenance of `ok`, `warn`, or `sink` classifications.
