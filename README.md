# Star Therapeutics

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
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Star Therapeutics is a clinical-stage biotechnology company in South San Francisco, California,
developing best-in-class antibody therapies in hematology and immunology under a hub-and-spoke
model. Each spoke pursues a distinct area of novel biology and advances a single antibody across
multiple related indications — a "pipeline-in-a-product" strategy. Its named spokes are Vega
Therapeutics (VGA039, a first-in-class anti-Protein S monoclonal antibody for von Willebrand
disease) and Electra Therapeutics (ELA-026, targeting SIRP proteins in secondary HLH). The company
emerged from stealth in February 2022, added $90M in 2023, and closed an oversubscribed $125M
Series D in September 2025; Incyte completed its acquisition of Vega Therapeutics in July 2026.

## API surface

**Star Therapeutics runs no developer program.** It publishes no product API, no developer portal,
no API documentation, no SDKs and no OpenAPI. The only machine-readable surface reachable without
credentials is the **WordPress REST content API** behind `star-therapeutics.com`, which serves the
company's 28-item news archive, 11 corporate pages and 661-item media library. The OpenAPI in this
repo is an API Evangelist derivation of the route index the site publishes at `/wp-json/`, verified
against live anonymous responses on 2026-08-05.

Notably, the site registers two agent-facing capability registries — a JetEngine MCP server at
`/wp-json/jet-engine/v1/mcp` and the WordPress Abilities API at `/wp-json/wp-abilities/v1/` — but a
JSON-RPC `tools/list` POST returns `401 rest_forbidden` and every abilities route returns 401. Both
are plugin defaults bound to an authenticated WordPress user, not an agent surface the company
publishes, so no MCP server is claimed for this provider. No agent card is served at either
`/.well-known/agent-card.json` or `/.well-known/agent.json` (both 404).

## Links

- https://star-therapeutics.com/
- https://star-therapeutics.com/news/
- https://www.linkedin.com/company/star-tx
- https://vegatherapeutics.com/ · https://electra-therapeutics.com/
- https://forgeglobal.com/star-therapeutics_stock/
