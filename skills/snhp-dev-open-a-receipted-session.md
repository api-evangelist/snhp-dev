---
generated: '2026-09-19'
method: generated
name: Open a $2 receipted negotiation session (key, wallet, session, receipt)
description: >-
  Mint a key, fund the prepaid wallet by Stripe Checkout (human) or MPP (agent-native), open a deterministic receipted session, take moves, close it, and verify the Ed25519 receipt offline.
api: openapi/snhp-dev-openapi.yml
operations: [issue_key_v1_keys_post, balance_v1_billing_balance_get, store_catalog_v1_store_catalog_get, checkout_session_v1_billing_checkout_session_post, mpp_manifest_v1_mpp_manifest_get, mpp_topup_v1_mpp_topup_post, open_advice_session_v1_advice_session_post, advice_move_v1_advice_move_post, advice_bundle_move_v1_advice_bundle_post, close_advice_session_v1_advice_close_post, store_notary_pubkey_v1_store_notary_pubkey_get]
source: >-
  Grounded in openapi/snhp-dev-openapi.yml (OpenAPI 3.1.0, captured live from
  https://snhp.dev/openapi.json); every operationId verified verbatim in the spec by the script that
  wrote this file. Flow and rules taken from https://snhp.dev/llms.txt, /.well-known/agents.json,
  GET /v1/store/catalog and the repository's A2A_FLOW.md / PRICING.md. No write was exercised.
---

# Open a $2 receipted negotiation session

## What you are buying
$2.00 once covers EVERY move of one negotiation — cap 10 moves, 7-day window — category-tuned
(`resale` | `supply` | `retail`), **deterministic** (same context in, bit-identical advice out,
auditable via `context_hash`) and **receipted** (Ed25519-signed per move). The free turn is the
unreceipted rehearsal of this.

## Money rules you must know BEFORE step 3
- Credit is **prepaid and non-refundable** — "there is no cashout path". Top up only what you need
  (the $2 custom minimum exists for that).
- Anchor SKUs (this session) charge the **full $2 up front**; an underfunded wallet gets a `402`
  with top-up options, never a partial charge.
- A paid call that FAILS is an uncharged `200 {ok:false, charged:false, reason, code}` — read
  `ok`, not the status code.
- Nothing here is idempotent: a retried `open` opens a second $2 session. On a timeout, read
  `balance_v1_billing_balance_get` (`GET /v1/billing/balance`) before retrying.

## Steps
1. **Mint a key** — `issue_key_v1_keys_post` (`POST /v1/keys`) `{agent_id, contact_email,
   intended_use_summary, telemetry_consent?}` → `{api_key: gt_..., rate_limit_per_minute: 600,
   wallet}`. Shown once; store it. The key arrives with a one-time **$0.50 starter credit** — a
   taste, not enough for a session.
2. **Read the shelf** — `store_catalog_v1_store_catalog_get` (`GET /v1/store/catalog`): `slots[]` with
   `negotiate.session` price 200000 millicents ($2.00), the counter fee (5% + 30¢ on top-ups), the
   `receipts` block (hash recipe, signing scheme, notary PEM + fingerprint).
3. **Fund the wallet** — one of:
   - human: `checkout_session_v1_billing_checkout_session_post` (`POST /v1/billing/checkout_session`) `{api_key,
     amount_cents}` → a Stripe Checkout URL a person opens once;
   - agent-native (MPP): `mpp_manifest_v1_mpp_manifest_get` (`GET /v1/mpp/manifest`) (pure read: fee, SPT
     minimum, the 402 flow), then `mpp_topup_v1_mpp_topup_post` (`POST /v1/mpp/topup`) with no credential →
     `402 application/problem+json` + `WWW-Authenticate: Payment ...`; authorise the challenge
     with a Stripe Shared Payment Token you minted scoped to this store; retry with
     `Authorization: Payment <credential>`; a `Payment-Receipt` header comes back. $2.00 of
     credit costs **$2.40** ($2.00 + 5% + $0.30). Crypto is declined.
4. **Open** — `open_advice_session_v1_advice_session_post` (`POST /v1/advice/session`) `SessionOpenIn`
   `{api_key, category, side, walk_away, target, their_offers[], my_offers[]?, rounds_left?}` →
   `{session_id, first_move: {offer, message, receipt}}`. The $2 is spent now.
5. **Move** — `advice_move_v1_advice_move_post` (`POST /v1/advice/move`) `SessionMoveIn` `{api_key, session_id,
   their_offers[] (FULL history, oldest first), my_offers[], rounds_left}` — no extra charge, up
   to the 10-move cap. Multi-issue inside the same session:
   `advice_bundle_move_v1_advice_bundle_post` (`POST /v1/advice/bundle`) `BundleMoveIn`.
6. **Close** — `close_advice_session_v1_advice_close_post` (`POST /v1/advice/close`) `{api_key, session_id}` →
   `closed` + a signed session-summary receipt (moves, total charged, per-move `context_hash`).
   Optional (sessions expire on their own); the one idempotent call in the flow.
7. **Verify offline** — `store_notary_pubkey_v1_store_notary_pubkey_get` (`GET /v1/store/notary_pubkey`) → pin
   `pubkey_pem` / `fingerprint`; recompute `signed_bytes` = canonical JSON of every receipt field
   except `signature`, base64-decode `signature`, Ed25519-verify. No call back to SNHP needed.

## Errors
`422` missing `api_key` (body) or `X-API-Key` (header) — the API says 422, not 401. `402` on an
underfunded anchor SKU or on the MPP challenge (expected). `429` + `Retry-After`.

## Notes
Key rotation (`rotate_key_v1_keys_rotate_post`) carries the balance over and kills the old key
**immediately, no grace** — do not rotate mid-session from a process that still holds the old key.
MCP twins: `session_open` / `session_advise` / `session_bundle` / `session_close` (key as a tool
argument). See `conventions/` (reversibility), `plans/`, `authentication/`.
