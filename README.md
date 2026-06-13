# DataForSEO

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
