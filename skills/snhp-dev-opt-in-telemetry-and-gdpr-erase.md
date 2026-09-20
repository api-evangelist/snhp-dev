---
generated: '2026-09-19'
method: generated
name: Opt in to telemetry, report an outcome, export and erase your rows (GDPR Art. 15/17)
description: >-
  Consent at key issuance, share a recommendation outcome within the same ISO week using the X-GT-Recommendation-Id, then export or delete every telemetry row tied to your key.
api: openapi/snhp-dev-openapi.yml
operations: [issue_key_v1_keys_post, negotiation_sell_next_offer_v1_negotiation_sell_next_offer_post, negotiation_buy_next_offer_v1_negotiation_buy_next_offer_post, telemetry_report_outcome_v1_telemetry_report_outcome_post, telemetry_export_v1_telemetry_export_get, telemetry_delete_v1_telemetry_delete_delete]
source: >-
  Grounded in openapi/snhp-dev-openapi.yml (OpenAPI 3.1.0, captured live from
  https://snhp.dev/openapi.json); every operationId verified verbatim in the spec by the script that
  wrote this file. Flow and rules taken from https://snhp.dev/llms.txt, /.well-known/agents.json,
  GET /v1/store/catalog and the repository's A2A_FLOW.md / PRICING.md. No write was exercised.
---

# Opt in to telemetry, report an outcome, export and erase (GDPR Art. 15/17)

## Default: nothing is collected
Two gates must BOTH be true for a row to be written: `telemetry_consent: true` at key issuance
(set ONCE, immutable) and `share_outcome: true` + an allowlisted `vertical` on the call.

## Steps
1. **Consent at issuance** — `issue_key_v1_keys_post` (`POST /v1/keys`) with `telemetry_consent: true`.
2. **Ask for a recommendation with sharing on** —
   `negotiation_sell_next_offer_v1_negotiation_sell_next_offer_post` (`POST /v1/negotiation/sell/next_offer`) or
   `negotiation_buy_next_offer_v1_negotiation_buy_next_offer_post` (`POST /v1/negotiation/buy/next_offer`) with `share_outcome: true`
   and `vertical` ∈ {ad_inventory, saas_procurement, cloud_compute, freight_logistics,
   media_licensing, m_and_a_buyside, m_and_a_sellside, real_estate, energy_trading,
   professional_services, marketplace_b2b, other}. The response carries
   `X-GT-Recommendation-Id: rec_*` — store it.
3. **Report the outcome** — `telemetry_report_outcome_v1_telemetry_report_outcome_post` (`POST /v1/telemetry/report_outcome`)
   `ReportOutcomeRequest` `{recommendation_id, deal_closed, my_utility, opponent_utility}` —
   **within the same ISO week** as the recommendation (the per-week key hash caps it at ~7 days).
4. **Access (Art. 15)** — `telemetry_export_v1_telemetry_export_get` (`GET /v1/telemetry/export`) → `{rows: [...]}`,
   every row tied to any of your week-hashes in the 78-week window.
5. **Erasure (Art. 17)** — `telemetry_delete_v1_telemetry_delete_delete` (`DELETE /v1/telemetry/delete`) →
   `{rows_deleted: N}`. "Sweeps the last 78 weeks of week-hashes." This is the ONE reversal with
   a stated window in the whole API. To stop future rows, also stop passing `share_outcome`.

## What is stored (provider's list)
A per-week HMAC of your key truncated to 128 bits (not reversible, not joinable across weeks);
numeric features quantised to a 0.02 grid; lists capped at 16; the vertical enum. NOT stored:
wall-clock timestamps, the raw key, free text, IPs, user agents.

## Notes
Applies "regardless of EU residence". Recorded in `regulatory/snhp-dev-regulatory-posture.yml`
as the provider's data-subject-request API.
