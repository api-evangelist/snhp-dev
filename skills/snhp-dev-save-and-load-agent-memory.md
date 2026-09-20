---
generated: '2026-09-19'
method: generated
name: Save and load encrypted agent memory across sessions (blind locker)
description: >-
  Park a ciphertext blob you encrypted yourself, receive a claim ticket and a signed receipt over its hash, and load it back free in any later session — the store never sees plaintext.
api: openapi/snhp-dev-openapi.yml
operations: [issue_key_v1_keys_post, memory_save_v1_memory_save_post, memory_load_v1_memory_parcel__ticket__get, store_notary_pubkey_v1_store_notary_pubkey_get]
source: >-
  Grounded in openapi/snhp-dev-openapi.yml (OpenAPI 3.1.0, captured live from
  https://snhp.dev/openapi.json); every operationId verified verbatim in the spec by the script that
  wrote this file. Flow and rules taken from https://snhp.dev/llms.txt, /.well-known/agents.json,
  GET /v1/store/catalog and the repository's A2A_FLOW.md / PRICING.md. No write was exercised.
---

# Save and load encrypted agent memory (blind locker)

## The contract
You encrypt BEFORE saving. The store holds ciphertext only, adds a second AES-256-GCM at-rest
layer, never decrypts or logs contents, and signs a receipt whose `content_hash` is blake2b-128 of
YOUR ciphertext. "A wrong owner is indistinguishable from a missing ticket."

## Auth and money
`gt_*` key (`issue_key_v1_keys_post` (`POST /v1/keys`)). Save is paid — flat park fee **$0.005 (≤64 KiB) or
$0.01 (≤256 KiB)** from the prepaid wallet, charged once and only after the blob is durably stored
(never on a failed/oversize park); the $0.50 starter credit covers the first saves. Load is free.

## Steps
1. **Encrypt locally** with a key that never leaves your host; base64 the ciphertext.
2. **Save** — `memory_save_v1_memory_save_post` (`POST /v1/memory/save`) `ParkIn` `{api_key, blob_b64,
   ttl_seconds?}` → `{ok, ticket, expires_at, size_bytes, receipt}`. TTL is clamped to
   **60 s – 604,800 s** (default 86,400 s); the effective `expires_at` is returned. Size cap
   262,144 bytes. (`POST /v1/store/park` is the same handler under its older name.)
3. **Keep the ticket.** There is no list operation; a lost ticket is a lost memory.
4. **Load** — `memory_load_v1_memory_parcel__ticket__get` (`GET /v1/memory/parcel/{ticket}`) with the key → `{ok, blob_b64,
   size_bytes, expires_at}`; decrypt locally. Past TTL → `expired`; missing/wrong owner → not
   found; `at_rest_key_unavailable` if the server's at-rest key is absent.
5. **Verify the receipt** against `store_notary_pubkey_v1_store_notary_pubkey_get` (`GET /v1/store/notary_pubkey`) the same
   way as any store receipt (see the session skill, step 7).

## Rules
- Save is NOT idempotent: a retried save parks (and charges for) a second blob. On a timeout,
  check the wallet before retrying.
- There is no delete; a memory disappears only by TTL. Choose `ttl_seconds` deliberately.
- Do not park plaintext. The provider cannot read it — and neither should a DB dump.

MCP twins: `memory_save` / `memory_load` (aliases `store_park` / `store_retrieve` on the pro door).
