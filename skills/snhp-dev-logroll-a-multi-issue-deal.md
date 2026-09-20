---
generated: '2026-09-19'
method: generated
name: Logroll a multi-issue deal with SNHP (free)
description: >-
  Propose a Pareto-efficient package across several linked issues — conceding what you value least to win what you value most — by giving per-option utilities per issue.
api: openapi/snhp-dev-openapi.yml
operations: [negotiate_bundle_endpoint_v1_negotiate_bundle_post]
source: >-
  Grounded in openapi/snhp-dev-openapi.yml (OpenAPI 3.1.0, captured live from
  https://snhp.dev/openapi.json); every operationId verified verbatim in the spec by the script that
  wrote this file. Flow and rules taken from https://snhp.dev/llms.txt, /.well-known/agents.json,
  GET /v1/store/catalog and the repository's A2A_FLOW.md / PRICING.md. No write was exercised.
---

# Logroll a multi-issue deal with SNHP (free)

## When to use
A deal with several linked issues that trade off (a job offer = base + equity + signing; a SaaS
contract = price + seats + term + SLA). Marked **beta** in `GET /v1/catalog`.

## Auth
None (60/min per IP), or a `gt_*` key header for 600/min — see the single-price skill.

## Steps
1. **Describe the issues** — `negotiate_bundle_endpoint_v1_negotiate_bundle_post` (`POST /v1/negotiate/bundle`), body
   `NegotiateBundleRequest`: `issues[]` of `BundleIssue` `{name, options[], my_utility[],
   their_utility[]}` (utilities per option in [0,1]; `their_utility` is your read of their
   DIRECTION), `their_offers[]` (packages they have tabled, oldest first), `my_priorities`
   (weights per issue), optional `my_batna`, `their_batna_estimate`, `rounds_left`.
2. **Read the package** — the recommended option per issue plus the trade logic (which issues
   were conceded to win which).
3. **Iterate** as their offers arrive; the engine infers their priorities from their offers.

## Honest caveat (the provider's, verbatim in /v1/catalog)
"the priority-inference is weak (r≈0.3) and adds only ~1% over a no-inference baseline — the
proven value is the efficient-package search, not yet the logrolling edge."

## Errors
`422` on malformed `issues[]` (every issue needs equal-length `options`, `my_utility`,
`their_utility`). `429` + `Retry-After` on the floor.

## Notes
- MCP twin: `negotiate_bundle`; the paid receipted version is `session_bundle` inside a $2 session.
- `score_deal` (MCP only, no REST twin) scores any package against the Pareto frontier.
