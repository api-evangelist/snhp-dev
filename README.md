# SNHP

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

SNHP (snhp.dev) is free, LLM-free game-theory negotiation math for AI agents — the math-optimal
next move in a single-price or multi-issue negotiation, auctions and mechanism design — served as
a REST API, a hosted MCP server (core and pro doors), a PyPI package, and behind an A2A agent card,
with a prepaid wallet (Stripe Checkout or the Machine Payments Protocol) for $2 receipted sessions
and encrypted agent memory. Operated by gametheory.dev / github.com/ryuxik. Surfaced by the
a2aregistry.org harvest of 2026-09-19 as "Negotiation Copilot for Agents (SNHP)".

- https://snhp.dev/
- https://snhp.dev/docs — Swagger UI over the live OpenAPI
- https://snhp.dev/llms.txt
- https://github.com/ryuxik/snhp

## What this profile holds

Profiled 2026-09-19. Every artifact below was searched, probed or derived from public surfaces —
see each file's `method:` and `source:` frontmatter. One free, keyless, documented quickstart call
(`POST /v1/negotiate/turn`) and one read-only MCP `tools/call` (`store_catalog`) were made; no
key was minted, no wallet funded, no paid or mutating operation exercised.

| Surface | Where |
|---|---|
| OpenAPI 3.1.0 — Game Theory Layer (74 operations, 78 schemas) and Evolution Arena (31 operations) | `openapi/` — verbatim originals in `openapi/_original/` |
| Proposed spec enhancements (servers, the two documented key schemes on 25 gated operations, 7 undeclared tags, the 429, the observed 402 problem+json) | `overlays/` |
| A2A agent card — served at the canonical path, **near-conformant** (no preferredTransport); its `url` is not an A2A endpoint | `a2a/` |
| Hosted MCP server — live, anonymous, core door 15 annotated tools + pro door 54; server card; official-registry entry | `mcp/` (raw `tools/list` for both doors + `initialize` saved) |
| MCP ↔ REST tool crosswalk (48 tools bound, 6 MCP-only, 38 of 74 operations reachable by tool; no key issuance over MCP) | `mcp/` |
| Six generated Agent Skills — free negotiate/bundle, the $2 receipted session with both funding rails, agent memory, verified-peer A2A + AP2 settlement, telemetry opt-in + GDPR erase | `skills/` |
| llms.txt (snhp.dev and arena.snhp.dev, provider-published) | `llms/` |
| `/.well-known/` probe across 6 hosts — agent card, MCP server card and agents.json served; nothing else | `well-known/` |
| Auth (documented but undeclared gt_* key; MPP payment challenge), conventions (idempotency partial, reversibility documented), errors, data model | `authentication/`, `conventions/`, `errors/`, `data-model/` |
| Plans (free core, $2 session, $0.005/$0.01 memory park, 5% + 30¢ top-up fee, one disputed line item), five published rate limits, best-effort SLA stated in the API, repository changelog | `plans/`, `rate-limits/`, `lifecycle/`, `changelog/` |
| No test mode — free production math, $0.50 starter credit, Swagger UI, synthetic-data dispute prototype | `sandbox/` |
| Packages — PyPI `snhp` 0.2.0 (library + stdio MCP server + API server); stale gametheory.dev URLs in its metadata | `packages/` |
| Domain security (TLS 1.3, HSTS, no SPF/DMARC/CAA/DNSSEC); no disclosure program or trust page found | `security/` |
| Horizontal regulatory posture — one signal, GDPR Art. 15/17 as API operations; no legal pages at all | `regulatory/` |
| Standards conformance incl. the two domain standards declared in the contract (MPP, AP2) and what is **not** conformant | `conformance/` |
| Recommended agentic-access execution contracts (generated, 105 operations) | `agentic-access/` |

Headline findings: the served OpenAPI declares no servers and no security while ~25 operations
require a key (a missing key answers 422, not 401); the A2A card advertises a REST + MCP surface
rather than an A2A endpoint (POST to its `url` → 405); exactly one write is idempotent (key
issuance, 24 h on `agent_id`); wallet credit is prepaid and non-refundable and the only windowed
reversal is telemetry erasure (78 weeks); and the provider publishes its prices, its SLA ("no
uptime SLA today — best-effort, single deployment"), its receipt-verification recipe and its GDPR
endpoints inside the API itself while serving no terms, privacy or security page.
