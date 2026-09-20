---
generated: '2026-09-19'
method: generated
name: Negotiate a single price with SNHP (free, no key)
description: >-
  Get the math-optimal counter-offer, a ready-to-send message and accept/walk advice for a multi-round single-price negotiation, in plain dollars, with no account.
api: openapi/snhp-dev-openapi.yml
operations: [negotiate_turn_endpoint_v1_negotiate_turn_post, issue_key_v1_keys_post]
source: >-
  Grounded in openapi/snhp-dev-openapi.yml (OpenAPI 3.1.0, captured live from
  https://snhp.dev/openapi.json); every operationId verified verbatim in the spec by the script that
  wrote this file. Flow and rules taken from https://snhp.dev/llms.txt, /.well-known/agents.json,
  GET /v1/store/catalog and the repository's A2A_FLOW.md / PRICING.md. No write was exercised.
---

# Negotiate a single price with SNHP (free, no key)

## When to use
Multi-round haggling over ONE price against ANY counterparty. Not for one-shot fixed prices (the
tool tells you so in `fit`), not for multi-issue packages (use the bundle skill), not for
non-price decisions.

## Auth
None. Base `https://snhp.dev`. Keyless callers share **60 requests/min per IP**. If you will call
in a loop, first mint a key with `issue_key_v1_keys_post` (`POST /v1/keys`) — body
`{agent_id, contact_email, intended_use_summary}`, no human approval, idempotent on `agent_id`
for 24 h — and send it as `Authorization: Bearer gt_...` for **600/min**. A key in the body does
not raise the limit.

## Steps
1. **Call the turn** — `negotiate_turn_endpoint_v1_negotiate_turn_post` (`POST /v1/negotiate/turn`), body
   `NegotiateTurnRequest`: `side` (`buy`|`sell`), `walk_away` (your true floor/ceiling in dollars),
   `target`, `counterparty_offers[]` (their offers so far, oldest first), optional
   `my_previous_offers[]`, `rounds_left`, `item`.
2. **Read the response** — `action` (`counter` | `accept` | `walk`), `recommended_price`,
   `message` (send it verbatim or feed it to your own LLM to restyle — SNHP hosts no LLM),
   `rationale`, `fit.score`, `expected_settlement`, `confidence`.
3. **Repeat each round** with the full offer history. The free turn is **non-deterministic**: the
   same input can return a different counter on a second call (documented example 5,386.6;
   observed 5,752.2). Do not compare two free runs as if they were one.
4. **Stop when** `action` is `accept` or `walk`. SNHP never sends the offer for you — "Auto-
   execution: never. We return recommendations; your environment delivers offers."

## Errors
- `422` `{"detail":[{loc, msg, type}]}` — a required field is missing (`side`, `walk_away`,
  `target`). See `errors/snhp-dev-problem-types.yml`.
- `429` with `Retry-After` seconds — sleep exactly that long. See `rate-limits/`.

## Notes
- The BATNA guard is enforced server-side: it will not recommend below your `walk_away`.
- Want a receipt, determinism and session state? That is the $2 session skill.
- Same math over MCP: tool `negotiate` at `https://snhp.dev/mcp/` (see `mcp/`).
