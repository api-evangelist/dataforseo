---
name: dataforseo-onpage-site-audit
description: >-
  Run a technical SEO crawl of a website with the DataForSEO OnPage API and pull
  the findings that matter — crawl summary, per-page issues, broken links,
  redirect chains, non-indexable pages, duplicate tags and content, and a
  Lighthouse performance audit. Use when asked to audit a site's technical SEO,
  find broken links or redirect chains, check indexability, or measure page
  performance. For search-ranking data use the SERP skill instead.
api: openapi/dataforseo-onpage-api-openapi.yml
operations:
  - TaskPost
  - OnPageTasksReady
  - Summary
  - Pages
  - Links
  - RedirectChains
  - NonIndexable
  - DuplicateTags
  - DuplicateContent
  - Resources
  - InstantPages
  - LighthouseLiveJson
  - ForceStop
  - OnPageAvailableFilters
  - OnPageErrors
generated: '2026-08-13'
method: generated
source: >-
  openapi/dataforseo-onpage-api-openapi.yml, conventions/dataforseo-conventions.yml,
  errors/dataforseo-problem-types.yml, sandbox/dataforseo-sandbox.yml
---

# DataForSEO — OnPage site audit

Base URL `https://api.dataforseo.com`. Rehearse free against
`https://sandbox.dataforseo.com` (same paths, same credentials, dummy data).

Auth is HTTP Basic — see `authentication/dataforseo-authentication.yml`.

OnPage is a **crawl**, not a lookup. You start a crawl once, wait for it, then query the
same crawl id repeatedly from a dozen different report endpoints. Do not start a new
crawl per report.

## Step 1 — Start the crawl

`POST /v3/on_page/task_post` (operationId **TaskPost**)

Body is a JSON array of task objects. Key fields: `target` (the domain), `max_crawl_pages`,
`start_url`, `enable_javascript`, `custom_js`, `respect_sitemap`, `crawl_delay`,
`load_resources`, `enable_browser_rendering`, plus `pingback_url` / `postback_url` and your
own `tag`.

Set `max_crawl_pages` deliberately — you are billed per page crawled.

Expect `status_code` **20100**. Keep the returned task `id`; every report below needs it.

## Step 2 — Wait

`GET /v3/on_page/tasks_ready` (operationId **OnPageTasksReady**) lists finished crawls,
or set a callback on the task. `40601` / `40602` mean the crawl is still queued or running.

`GET /v3/on_page/summary/{id}` (operationId **Summary**) is the progress-and-overview
endpoint: it returns crawl progress, pages crawled/in queue, and the aggregate
`page_metrics` — counts of broken links, duplicate titles, non-indexable pages,
checks failed, and the OnPage Score.

Poll `Summary` until `crawl_progress` is `finished`. To abandon a crawl early, use
`POST /v3/on_page/force_stop` (operationId **ForceStop**).

## Step 3 — Pull the reports

All of these are POST, all take the crawl `id`, and all accept `limit`, `offset`,
`filters` and `order_by`:

| Report | Operation | Path |
|---|---|---|
| Per-page results and checks | **Pages** | `POST /v3/on_page/pages` |
| Pages grouped by a resource | **PagesByResource** | `POST /v3/on_page/pages_by_resource` |
| Images, scripts, stylesheets | **Resources** | `POST /v3/on_page/resources` |
| Internal/external links, broken links | **Links** | `POST /v3/on_page/links` |
| Redirect chains | **RedirectChains** | `POST /v3/on_page/redirect_chains` |
| Pages blocked from indexing | **NonIndexable** | `POST /v3/on_page/non_indexable` |
| Duplicate title/description tags | **DuplicateTags** | `POST /v3/on_page/duplicate_tags` |
| Duplicate body content | **DuplicateContent** | `POST /v3/on_page/duplicate_content` |
| Load-time waterfall for a page | **Waterfall** | `POST /v3/on_page/waterfall` |
| Keyword density | **KeywordDensity** | `POST /v3/on_page/keyword_density` |
| Structured data found | **Microdata** | `POST /v3/on_page/microdata` |
| Raw HTML of a crawled page | **RawHtml** | `POST /v3/on_page/raw_html` |
| Screenshot of a page | **PageScreenshot** | `POST /v3/on_page/page_screenshot` |
| Parsed readable content | **ContentParsing** | `POST /v3/on_page/content_parsing` |

Before writing a `filters` array, fetch the legal fields and operators for the endpoint:
`GET /v3/on_page/available_filters` (operationId **OnPageAvailableFilters**). Guessing a
filter field returns `40501` (invalid field) or `40506` (unknown fields in POST data).

## Auditing a handful of URLs without a crawl

`POST /v3/on_page/instant_pages` (operationId **InstantPages**) returns full OnPage checks
for specific URLs synchronously — no crawl id, no waiting. Use it when the ask is "check
these three pages", not "audit this site".
`POST /v3/on_page/content_parsing/live` (**ContentParsingLive**) is the synchronous
content-extraction sibling.

## Performance audit

`POST /v3/on_page/lighthouse/live/json` (operationId **LighthouseLiveJson**) runs
Lighthouse synchronously and returns the JSON report. For volume, use
`POST /v3/on_page/lighthouse/task_post` (**LighthouseTaskPost**) →
`GET /v3/on_page/lighthouse/tasks_ready` (**LighthouseTasksReady**) →
`GET /v3/on_page/lighthouse/task_get/json/{id}` (**LighthouseTaskGetJson**).
Available audits and versions: **LighthouseAudits**, **LighthouseVersions**,
**LighthouseLanguages**.

## Failure handling

Same envelope rules as everywhere in DataForSEO: HTTP 200 with a numeric `status_code`.
The ones specific to OnPage:

| status_code | Meaning |
|---|---|
| `40406` | Requested page was not submitted for crawling — it is not in this crawl id. |
| `40405` | Target page has insufficient textual content. |
| `40408` | Target URL is invalid or returned 404. |
| `40407` | Duplicate host in the request. |
| `50402` | Target page took longer than 50 seconds to respond. |

`POST /v3/on_page/errors` (operationId **OnPageErrors**) lists OnPage tasks that errored in
the past 7 days.

Crawl results are retained for 30 days; after that `Summary` and the report endpoints
return `40403` and the crawl must be re-run. There is no idempotency key — re-posting a
crawl re-crawls and re-charges, so check `POST /v3/on_page/id_list` (**OnPageIdList**)
before retrying.
