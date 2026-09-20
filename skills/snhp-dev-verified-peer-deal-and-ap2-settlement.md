---
generated: '2026-09-19'
method: generated
name: Verified-peer A2A deal with AP2 settlement (both agents run SNHP)
description: >-
  Register an Ed25519 operator identity (optionally domain-verified), exchange locally signed peer proofs, open a session whose peer_mode the server derives, negotiate under it, and settle into a signed AP2 Cart Mandate.
api: openapi/snhp-dev-openapi.yml
operations: [register_operator_v1_registry_register_operator_post, request_domain_challenge_v1_registry_request_domain_challenge_post, verify_domain_v1_registry_verify_domain_post, open_session_v1_a2a_open_session_post, next_offer_v1_a2a_next_offer_post, settle_v1_a2a_settle_post, keys_settlement_notary_v1_keys_settlement_notary_get]
source: >-
  Grounded in openapi/snhp-dev-openapi.yml (OpenAPI 3.1.0, captured live from
  https://snhp.dev/openapi.json); every operationId verified verbatim in the spec by the script that
  wrote this file. Flow and rules taken from https://snhp.dev/llms.txt, /.well-known/agents.json,
  GET /v1/store/catalog and the repository's A2A_FLOW.md / PRICING.md. No write was exercised.
---

# Verified-peer A2A deal with AP2 settlement

## Only when BOTH sides run SNHP
Against an unknown counterparty there is nothing to verify — use the free negotiate skills. This
flow is what the agent card's `capabilities.extensions[0]` (uri
`https://snhp.dev/a2a/negotiation/v1`) advertises. It is **not** an A2A JSON-RPC endpoint: every
step is an ordinary REST POST (plus one local MCP-only signing step).

## Once per operator
1. Generate an Ed25519 keypair locally.
2. **Register** — `register_operator_v1_registry_register_operator_post` (`POST /v1/registry/register_operator`)
   `RegisterOperatorRequest` `{operator_id, public_key_b64, display_name?}` →
   `{attestation_jwt, verification_level: "self"}`.
3. **Optional domain upgrade** — `request_domain_challenge_v1_registry_request_domain_challenge_post` (`POST /v1/registry/request_domain_challenge`)
   `{domain, public_key_b64}` → a DNS-TXT record to publish; then
   `verify_domain_v1_registry_verify_domain_post` (`POST /v1/registry/verify_domain`) `{domain, public_key_b64, display_name?}`
   → `verification_level: "domain"` (sybil-resistant; a counterparty can require it).

## Per deal
4. **Build a peer proof LOCALLY** — MCP tool `gt_a2a_build_peer_proof` on the pro door
   (`https://snhp.dev/mcp/pro/`) or the `snhp` stdio server on your own host: binds
   `attestation_jwt` + `operator_id` to THIS `negotiation_id` and `role` with your PRIVATE key,
   short-lived. There is deliberately no REST twin — the key never leaves your process.
5. **Exchange proofs** over your own channel (or as an A2A message Part).
6. **Open the session** — `open_session_v1_a2a_open_session_post` (`POST /v1/a2a/open_session`) `OpenSessionRequest`
   `{negotiation_id, seller_proof: PeerProof, buyer_proof: PeerProof, require_level?}` →
   `{session_id, peer_mode, self_deal}`. `peer_mode` is **server-derived**: true only if both
   proofs verify at/above `require_level`, match their roles, are unexpired and name DISTINCT
   operators. You cannot claim it by asserting it.
7. **Negotiate** — `next_offer_v1_a2a_next_offer_post` (`POST /v1/a2a/next_offer`) `NextOfferRequest` `{session_id,
   role, my_reservation, opponent_offer_history[], my_offer_history[], deadline_rounds?,
   pareto_knob?}` — NORMALISED utility in [0,1], not dollars (map the way `negotiate` does).
8. **Settle** — `settle_v1_a2a_settle_post` (`POST /v1/a2a/settle`) `SettleRequest` `{session_id, agreed_price,
   currency, item?, buyer_max_price?, terms?}` → `{cart_mandate: <AP2 VC-JWT>}` (+
   `intent_mandate` when `buyer_max_price` is passed). Refused unless `peer_mode` is true.
9. **Verify** the mandate against `keys_settlement_notary_v1_keys_settlement_notary_get` (`GET /v1/keys/settlement_notary`)
   (Ed25519 PEM, also published in the agent card as `settlement_notary_public_key_pem`).

## What settlement is and is not
"No escrow; the mandate is the settlement" — a signed, non-repudiable RECORD naming two verified
parties, not a money movement. There is no void/cancel operation (see `conventions/` reversibility).
The provider prices no fee on it today (PRICING.md withdraws the earlier 0.1–0.5%; `GET
/v1/catalog` still shows it — see `plans/`).

## Errors
`422` on missing fields; a forged, expired, below-level or self-dealing proof yields
`peer_mode=false` rather than an error. `429` + `Retry-After`.
