# DataForSEO

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

DataForSEO is an SEO data platform offering REST APIs for SERP results, keyword data, backlink analysis, on-page audits, domain analytics, and competitor research across 100+ search engines. It provides pay-as-you-go access to structured SEO and digital marketing data for software developers, agencies, and enterprise teams.

## APIs

| API | Description |
|-----|-------------|
| SERP API | Search engine results pages across Google, Bing, YouTube, Yahoo, Baidu, Naver, and Seznam |
| Keywords Data API | Search volume, CPC, and keyword metrics from Google Ads, Bing Ads, and Google Trends |
| Backlinks API | Real-time structured backlink data for any domain, subdomain, or page |
| DataForSEO Labs API | Advanced search analytics, keyword research, and competitor intelligence |
| Domain Analytics API | Technology detection, traffic estimates, and WHOIS data for any domain |
| On-Page API | 100+ on-page SEO parameters via website crawling and technical audit |
| Content Analysis API | Brand monitoring, sentiment analysis, and citation tracking |
| Business Data API | Customer reviews and business data from Google, Trustpilot, TripAdvisor |
| App Data API | App rankings, reviews, and metadata from Google Play and Apple App Store |
| Merchant API | Product and pricing data from Google Shopping and Amazon |
| AI Optimization API | LLM mentions tracking and generative engine optimization data |
| Social Media API | Social engagement data for Pinterest and Facebook |

## Authentication

DataForSEO APIs use HTTP Basic Authentication. Credentials (login and password) are provided upon account registration at [app.dataforseo.com](https://app.dataforseo.com).

## Pricing

DataForSEO operates on a **pay-as-you-go** model with no monthly subscription fees. A minimum prepaid deposit of $50 is required to activate an account.

| Queue | Price per SERP | Latency |
|-------|---------------|---------|
| Standard Queue | $0.0006 | ~5 minutes |
| Priority Queue | $0.0012 | ~1 minute |
| Live Mode | $0.0020 | ~6 seconds |

See [plans/dataforseo-plans-pricing.yml](plans/dataforseo-plans-pricing.yml) for full pricing details.

## Rate Limits

- Maximum: **2,000 API calls per minute** per account
- Rate limit status is returned via `X-RateLimit-Limit` and `X-RateLimit-Remaining` response headers

See [rate-limits/dataforseo-rate-limits.yml](rate-limits/dataforseo-rate-limits.yml) for details.

## SDKs

- [Python](https://github.com/dataforseo/PythonClient)
- [TypeScript](https://github.com/dataforseo/TypeScriptClient)
- [Java](https://github.com/dataforseo/JavaClient)
- [C#](https://github.com/dataforseo/CSharpClient)
- [MCP Server (TypeScript)](https://github.com/dataforseo/mcp-server-typescript)
- [n8n Node](https://github.com/dataforseo/n8n-nodes-dataforseo)

## OpenAPI

Full OpenAPI documentation for all DataForSEO endpoints is available at [github.com/dataforseo/OpenApiDocumentation](https://github.com/dataforseo/OpenApiDocumentation).

## Links

- **Website:** https://dataforseo.com
- **Documentation:** https://docs.dataforseo.com
- **GitHub:** https://github.com/dataforseo
- **Pricing:** https://dataforseo.com/pricing
- **Status:** https://status.dataforseo.com
- **Blog:** https://dataforseo.com/blog
- **LinkedIn:** https://www.linkedin.com/company/dataforseo
- **X:** https://x.com/dataforseo

## Maintainer

Kin Lane — kin@apievangelist.com
