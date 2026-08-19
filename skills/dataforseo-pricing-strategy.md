---
name: pricing-strategy
description: Analyzes pricing across all sellers for a single Amazon ASIN and recommends pricing positioning when the user provides their own product ASIN or a competitor ASIN and asks to "set my price", "price this product", "compete on price", "win the Buy Box", "see who's got the Buy Box", "check all sellers", "what should I price at", or "pricing strategy for [ASIN]". Pulls every seller offering the ASIN, identifies the Buy Box winner, maps the price range across new and used conditions, and returns a target price plus a positioning rationale. Do not use for broad category competitor analysis across multiple products — that is the amazon-competitor-auditor skill's job.
---

# Pricing Strategy

You are giving a seller a defensible, data-backed price for a specific Amazon product (their own or a competitor's). Produce a single markdown report in chat that ends with a specific target price and a positioning rationale.

## Inputs to confirm before any tool call

Ask the user (in one short message) for whichever of these are not already clear from context:

1. **ASIN** — the 10-character Amazon identifier of the product to analyze. Required. If the user pastes a full Amazon URL, extract the ASIN from the `/dp/<ASIN>/` segment.
2. **Marketplace** — default to "United States" but always confirm. Common: United States, United Kingdom, Germany, Canada, Australia, France, Italy, Spain, Japan, India, Mexico.
3. **Whose product is this?** — ask whether the ASIN is theirs (they want a price for their own listing) or a competitor's (they want positioning intel). This changes the framing of the recommendation but not the data pulled.
4. **Their cost floor (optional but recommended when it's their own product)** — if they're pricing their own listing, ask for their unit cost or minimum acceptable price. If they give one, no target price recommendation should fall below it.

## Workflow

### Step 0 — Verify the DataForSEO connection (preflight)

Run this before anything else. Do not skip it, even if the user seems to expect immediate results.

1. **Check tool availability.** Confirm the DataForSEO Amazon merchant tools are in your available tool list — specifically `mcp__dataforseo__merchant_amazon_locations`, `mcp__dataforseo__merchant_amazon_asin_live_advanced`, and `mcp__dataforseo__merchant_amazon_sellers_live_advanced`. (The exact prefix is `mcp__dataforseo__` for the standard DataForSEO MCP; if the user named their connection differently, the suffix `merchant_amazon_locations` still applies.) If no DataForSEO Amazon tools are available at all, stop and tell the user:

   > "I can't run this — the DataForSEO MCP isn't connected. To connect it: open the connector picker (the connectors/tools menu), search for 'DataForSEO', and complete the setup with your DataForSEO API credentials. Once it's connected, ask me again."

   Do not attempt any analysis. Do not fabricate data.

2. **Test the connection with a free call.** If the tools are available, call `mcp__dataforseo__merchant_amazon_locations` with `country: "US"` and `limit: 1`. This is a free utility endpoint — it consumes no paid API credits.
   - If it returns `status_code == 20000`, the connection is healthy. Proceed silently to Step 1 — do not announce the check passed unless the user asked for a connection test.
   - If it returns a non-20000 `status_code` or errors, the MCP is connected but credentials are failing. Stop and report the `status_message` to the user, and tell them to verify their DataForSEO API login/credentials in the connector settings.

### Step 1 — Pull product context

Call `mcp__dataforseo__merchant_amazon_asin_live_advanced` with the ASIN and marketplace. Extract:
- `items[0].title`
- `items[0].data_asin`
- `items[0].price_from`, `items[0].price_to`, `items[0].percentage_discount`, `items[0].currency`
- `items[0].rating.value`, `items[0].rating.votes_count`
- `items[0].is_amazon_choice`, `items[0].is_available`
- `items[0].product_information[]` — find `section_name == "Item details"` and read `body["Brand Name"]` and `body["Best Sellers Rank"]`

This gives the product identity and the headline price Amazon displays.

### Step 2 — Pull every seller

Call `mcp__dataforseo__merchant_amazon_sellers_live_advanced` with the same ASIN and marketplace. Response `items[]` shape:

- `items[0]` has `type == "amazon_seller_main_item"` — **this is the Buy Box winner**. There is exactly one.
- Remaining items have `type == "amazon_seller_item"` — other offers.

Per seller, extract:
- `seller_name`
- `ships_from` — use this to detect fulfillment: if it equals or contains `"Amazon"`, this seller is FBA/Amazon-fulfilled.
- `price.current` (the offer price), `price.regular` (if discounted), `percentage_discount` (if present), `price.currency`
- `condition` — `"New"`, `"Used - Like New"`, `"Used - Very Good"`, `"Used - Acceptable"`, etc.
- `rating.value` and `rating.votes_count` (seller reputation, not product rating; may be absent for Amazon itself)
- `delivery_info.delivery_message` and `delivery_info.delivery_price.current` (delivery surcharge, if any)

### Step 3 — Compute the pricing landscape

Compute these separately for **New** and **Used** conditions (segment by `condition` starting with `"New"` vs `"Used"`):

- **New min, median, max** of `price.current`
- **Used min, median, max** of `price.current` (skip this section if no used offers)
- **Buy Box winner's price** — `items[0].price.current`
- **Buy Box winner's fulfillment** — FBA (ships from Amazon) vs FBM (ships from seller)
- **Delta from Buy Box** — for each non-winning seller, compute `price.current - buy_box_price`. Identify the closest competitor (smallest positive delta) and the cheapest seller overall.
- **Sellers with shipping surcharges** — flag any with `delivery_info.delivery_price.current > 0`; their effective total is higher than the listed price.

### Step 4 — Decide the recommendation

This skill returns **both** a specific target price and a positioning rationale.

**Target price logic** (apply in order, stop at the first rule that fits):

1. **If the user is pricing their own listing AND gave a cost floor**: target the higher of (a) Buy Box price minus 1% rounded to the nearest $0.99 ending, or (b) cost floor + the smallest defensible margin. If (b) > (a), say so honestly: "to undercut the Buy Box you would need to price below your cost floor — at your floor, you sit $X above the Buy Box and rely on [reputation / Prime / reviews] instead."
2. **If the user is pricing their own listing with no cost floor given**: target Buy Box price − 1% rounded to $0.99 if the seller is FBA, or Buy Box price − 3% if the Buy Box winner is FBM (FBM is easier to outcompete on raw price because Amazon weights FBA heavily for the Buy Box). Cap the target at the New median to avoid a race to the bottom.
3. **If the user is analyzing a competitor's ASIN**: target the New median minus 2–5% as a "competitive entry" price, and explicitly note that the Buy Box currently sits at $[buy_box_price] and would be the threshold to beat.

**Positioning rationale** — always include, regardless of which rule fired:

- Where the target price sits in the New band (bottom 25%, near median, top 25%).
- Who they're competing with at that price tier.
- Whether the Buy Box is realistically winnable from that target (FBA winners with high seller ratings are hard to dislodge; FBM winners or low-rated sellers are easier).
- One sentence on what the seller would need *besides* price to convert at that target — Prime eligibility, seller rating threshold, review velocity, etc.

### Step 5 — Build the report in chat

Output this structure directly in chat. No file creation.

```
# Pricing analysis — [product title]

**ASIN:** `[ASIN]` · **Marketplace:** [location_name] · **Brand:** [Brand Name] · **Product rating:** [rating] ([votes] reviews)

## Buy Box

**Winner:** [seller_name] — **$[price]** · [FBA or FBM] · ships from [ships_from]
[If the winner is Amazon.com itself, say so explicitly — that's the hardest Buy Box to take.]

## All sellers — New condition

| # | Seller | Price | Δ vs Buy Box | Fulfillment | Seller rating | Delivery note |
|---|---|---|---|---|---|---|
| 1 (Buy Box) | [name] | $X.XX | — | FBA/FBM | X.X ([votes]) | [delivery_message trimmed] |
| 2 | [name] | $X.XX | +$X.XX | FBA/FBM | ... | ... |

## All sellers — Used condition

[Same table, only if used offers exist. Otherwise: "No used offers listed."]

## The pricing landscape

- **New price range:** $[min] – $[max] (median $[med]) across [N] sellers
- **Used price range:** $[min] – $[max] (median $[med]) across [N] sellers, if applicable
- **Cheapest new offer:** [seller] at $[price] — [is it the Buy Box? if not, why not — e.g. FBM, low seller rating, delivery fee?]
- **Sellers with shipping surcharges:** [list, or "none"]

## Recommended price

**Target: $X.XX**

[2–4 sentences: why this number specifically — which rule from the logic fired, where it sits in the band, who else is at that tier. Reference the table above.]

## Positioning rationale

[2–4 sentences explaining the strategic position the target price puts the seller in. Cover: realism of winning the Buy Box from here, what else is needed besides price (Prime, ratings, review velocity), and what the next price-down move would look like if this target doesn't convert. If the user gave a cost floor, end with the margin implication at this target.]
```

## Rules

- Always show both the specific target price and the qualitative positioning rationale — never just one.
- All money references must include the currency symbol and match the marketplace's currency from the API response. Do not assume USD.
- Round target prices to a `.99` or `.95` ending — that's standard Amazon pricing convention and looks credible.
- Do not recommend a target below a stated cost floor. If the math points there, surface the conflict honestly.
- If the seller list has only one entry (the Buy Box winner with no competition), say so plainly and recommend pricing at or just below their level — there is no field to position against.

## Error handling

- If `merchant_amazon_asin_live_advanced` returns `status_code != 20000` or empty `items`, the ASIN is invalid — ask the user to verify it. Do not call sellers.
- If `merchant_amazon_sellers_live_advanced` returns `status_code != 20000`, surface the error and stop.
- If sellers returns zero items, the product has no current offers (out of stock everywhere). Report this and skip the price recommendation — there's nothing to position against.

## References

For the exact DataForSEO response shapes and field paths used above, see `references/dataforseo-shapes.md`.
