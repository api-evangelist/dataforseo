---
name: amazon-competitor-auditor
description: Audits Amazon competitors and produces a strategic markdown report when the user provides a competitor ASIN or a product keyword and asks to "audit competitors", "analyze competitors", "see what I'm up against", "check the competition for X", "who's winning on Amazon for Y", "compare competing products", or similar. Pulls the top 10 competing products, ranks them by price, rating, review count, and demand signals, identifies the category winners and why they're winning, and ends with strategic recommendations. Do not use for pricing positioning of a single product across sellers — that is the pricing-strategy skill's job.
---

# Amazon Competitor Auditor

You are auditing the competitive field on Amazon for a seller. Produce a single, decision-ready markdown report in chat — no files, no spreadsheets unless the user asks.

## Inputs to confirm before any tool call

Ask the user (in one short message) for whichever of these are not already clear from context:

1. **Target** — a competitor ASIN (10 characters, usually starts with `B0`) **or** a product keyword they sell against. If they give an ASIN, you'll use it to derive a category keyword (see step 1 below).
2. **Marketplace** — default to "United States" but always confirm. Common options: United States, United Kingdom, Germany, Canada, Australia, India, Japan, Mexico, Spain, France, Italy. If the user names a country, map it to the matching `location_name` string used by DataForSEO (e.g. "UK" → "United Kingdom", "DE" → "Germany").
3. **Their own ASIN (optional)** — if the user is auditing for their own product, ask whether they want it called out in the comparison. If they provide it, include their listing as a labeled row even if it isn't in the top 10.

Do not ask about "depth" — this skill always pulls and analyzes the top 10 organic competitors. Mention that fact in your confirmation message so the user knows.

## Workflow

### Step 0 — Verify the DataForSEO connection (preflight)

Run this before anything else. Do not skip it, even if the user seems to expect immediate results.

1. **Check tool availability.** Confirm the DataForSEO Amazon merchant tools are in your available tool list — specifically `mcp__dataforseo__merchant_amazon_locations` and `mcp__dataforseo__merchant_amazon_products_live_advanced`. (The exact prefix is `mcp__dataforseo__` for the standard DataForSEO MCP; if the user named their connection differently, the suffix `merchant_amazon_locations` still applies.) If no DataForSEO Amazon tools are available at all, stop and tell the user:

   > "I can't run this — the DataForSEO MCP isn't connected. To connect it: open the connector picker (the connectors/tools menu), search for 'DataForSEO', and complete the setup with your DataForSEO API credentials. Once it's connected, ask me again."

   Do not attempt any analysis. Do not fabricate data.

2. **Test the connection with a free call.** If the tools are available, call `mcp__dataforseo__merchant_amazon_locations` with `country: "US"` and `limit: 1`. This is a free utility endpoint — it consumes no paid API credits.
   - If it returns `status_code == 20000`, the connection is healthy. Proceed silently to Step 1 — do not announce the check passed unless the user asked for a connection test.
   - If it returns a non-20000 `status_code` or errors, the MCP is connected but credentials are failing. Stop and report the `status_message` to the user, and tell them to verify their DataForSEO API login/credentials in the connector settings.

### Step 1 — Resolve the search keyword

- If the user gave an **ASIN**: call `mcp__dataforseo__merchant_amazon_asin_live_advanced` with that ASIN and the marketplace location. From the response, derive the search keyword from the product title — pick the 2–4 most category-defining words (e.g. for "Owala FreeSip Insulated Stainless Steel Water Bottle..." use `stainless steel water bottle`, not the brand). Show the chosen keyword to the user in one line ("Searching the field for: `stainless steel water bottle`") before running the next call. The product details from this call are also kept aside for use in the final report.
- If the user gave a **keyword**: use it directly. Trim quotes and lowercase it.

### Step 2 — Pull the competitor field

Call `mcp__dataforseo__merchant_amazon_products_live_advanced` with:
- `keyword`: the resolved keyword
- `location_name`: the confirmed marketplace
- `language_code`: pick the natural language for that marketplace (`en_US`, `en_GB`, `de_DE`, `fr_FR`, `it_IT`, `es_ES`, `es_MX`, `ja_JP`, `en_IN`, etc.)
- `sort_by`: `"relevance"` (omit unless the user specifically asks for price- or rating-sorted results)

### Step 3 — Filter to organic competitors

From the response, take `items[]` and **keep only items where `type == "amazon_serp"`**. Discard `amazon_paid` (sponsored) and `related_searches`. Sort the kept items by `rank_absolute` ascending and take the first 10. These are the top 10 organic competitors.

For each, extract:
- `data_asin` — the ASIN
- `title` — trim to ~70 chars for the table
- `price_from` — current price (note: this can be missing on bundles/multi-packs; treat null as "—")
- `currency`
- `rating.value` and `rating.votes_count`
- `bought_past_month` — Amazon's "X bought past month" badge; null if absent
- `is_amazon_choice` — boolean
- `url` — full Amazon product URL

### Step 4 — Compute the comparative scores

Across the top 10:
- **Price band**: min, median, max of `price_from` (skip nulls). Note who is at each extreme.
- **Rating leader**: highest `rating.value`. Tie-break by `votes_count` (more reviews wins).
- **Review-count leader**: highest `rating.votes_count` — this is the social-proof incumbent.
- **Demand leader**: highest `bought_past_month` — this is the velocity signal.
- **Amazon's Choice badge holders**: list any items where `is_amazon_choice` is true.
- **Best-rated-with-meaningful-reviews**: highest `rating.value` among items with `votes_count >= 100`. This filters out fluke 5-star listings with 3 reviews.

### Step 5 — Build the report in chat

Output a single markdown report directly in chat. No file creation. Use this structure exactly:

```
# Amazon competitor audit — [keyword or product]

**Marketplace:** [location_name] · **Searched:** `[keyword]` · **Field size analyzed:** top 10 organic results

## The field

| # | Product | ASIN | Price | Rating | Reviews | Bought / mo | Badges |
|---|---|---|---|---|---|---|---|
| 1 | [title trimmed]([url]) | `[ASIN]` | $X.XX | 4.X (X,XXX) | XX,XXX | XX,XXX | Amazon's Choice |
| ... |

## Who's winning, and why

**Demand leader:** [title] — [bought_past_month] bought last month, [rating] stars on [votes] reviews, priced at $[price]. [One sentence of why they're likely winning — price position, social proof depth, badge, etc.]

**Rating leader (with real review depth):** [title] — [rating] on [votes] reviews. [What this signals.]

**Price floor:** [title] at $[price]. [Cheapest serious option; note if it's also rated well or if it's a bargain-bin entry.]

**Price ceiling:** [title] at $[price]. [Premium positioning — what justifies it: reviews, badge, brand?]

**Amazon's Choice:** [list of badge holders, or "None among the top 10."]

## Patterns in the field

[2–4 sentences describing structural patterns: is the price band tight or wide? Are the winners all in one price tier? Is there a clear premium tier and budget tier? Is there a gap a new entrant could exploit? Are review counts heavily lopsided toward one incumbent?]

## Strategic recommendations

1. **[Action verb] [specific move].** [One sentence justifying it from the data above. Cite specific prices, ratings, or ASINs.]
2. **[Action verb] [specific move].** [Justified from data.]
3. **[Action verb] [specific move].** [Justified from data.]

[If the user provided their own ASIN, add a fourth recommendation specifically about how their listing compares — where they sit on price, rating, review count, and the single highest-leverage change they could make.]
```

## Rules for the recommendations section

- Be specific. "Lower your price by ~10%" beats "consider pricing changes." Reference actual dollar amounts from the field.
- Tie every recommendation to evidence in the table above it. The user should be able to point at a row and see why you said what you said.
- Three recommendations is the floor; add a fourth only when the user provided their own ASIN.
- Do not recommend things you cannot verify from the data — e.g. don't speculate about ad spend, supply chain, or off-Amazon marketing.

## Error handling

- If the products call returns `status_code != 20000`, surface the `status_message` and stop. Do not fabricate competitors.
- If fewer than 10 organic results come back, work with what's there and note the small field size in the report header.
- If the user-provided ASIN returns `status_code != 20000` or the response has no `items`, tell the user the ASIN couldn't be resolved and ask them to verify it or switch to a keyword.

## References

For the exact DataForSEO response shapes and field paths used above, see `references/dataforseo-shapes.md`.
