# Automate Google Trends: Turn Search Interest Into a Trading Signal

Google Trends only shows you 5 terms at a time and no bulk export for serious monitoring. The `apify/google-trends-scraper` (16k users) turns Trends into an API — batch keywords, historical data, regional breakdowns. Here's the pipeline for content timing, niche research, and trend arbitrage.

> 🤖 **Try it live** — chat with a Poe bot that generates the exact pipeline for your keywords: [SERP Rank Tracker](https://poe.com/serpranktracker) · [Lead List Generator](https://poe.com/leadlistgenerator) · [Job Postings Miner](https://poe.com/jobpostingsminer) · [Amazon Product Miner](https://poe.com/amazonproductminer) · [Social Lead Finder](https://poe.com/socialleadfinder)

## The stack

- **[Apify](https://console.apify.com/sign-up?fpr=dx06p)** — `apify/google-trends-scraper` handles batching + export. Free monthly credits.
- **Python** — interest-over-time → alerts.

## Step 1 — Token

[Create an Apify account](https://console.apify.com/sign-up?fpr=dx06p) → Settings → Integrations → `export APIFY_TOKEN=...`

## Step 2 — Batch keyword interest

```bash
curl -X POST "https://api.apify.com/v2/acts/apify~google-trends-scraper/run-sync-get-dataset-items?token=$APIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "searchTerms": ["ai agent", "claude code", "cursor ide"],
    "timeRange": "today 12-m",
    "geo": "US",
    "isMultiple": true
  }'
```

Output: per-term time series + regional interest + related queries (`rising` = breakouts).

## Step 3 — Detect breakouts

```python
from apify_client import ApifyClient

client = ApifyClient("APIFY_TOKEN")
run = client.actor("apify/google-trends-scraper").call(run_input={
    "searchTerms": ["your_niche_term"], "timeRange": "today 3-m", "isMultiple": True,
})

for item in client.dataset(run["defaultDatasetId"]).iterate_items():
    for q in item.get("relatedQueries", {}).get("rising", []):
        print("BREAKOUT query:", q["query"], "| +", q.get("value"), "%")
```

`rising` queries marked "Breakout" = demand spiking right now — write the article, launch the product, buy the keyword before everyone else.

## The plays

- **Content sites** — rising queries → articles published before the wave = free traffic.
- **E-commerce** — regional interest map = where to test ads first.
- **Seasonality** — 5-year data → know exactly when demand peaks (e.g. "standing desk" spikes every January).

## Useful actors

| Actor | Job | Users |
|---|---|---|
| `apify/google-trends-scraper` | Batch trends, regions, related queries | 16.2k |
| `data_xplorer/google-trends-fast-scraper` | Fast trend pulls | 4.8k |
| `agenscrape/google-trends-scraper` | Interest/regions/queries | 1.4k |

Combine with `apify/google-autocomplete-scraper` → keyword ideas → trends validation → SERP scrape (`apify/google-search-scraper`) → full demand pipeline on [free credits](https://console.apify.com/sign-up?fpr=dx06p).

## More resources

- [apify-skills](https://github.com/g4dr/apify-skills) — `apify-seo-serp` skill: trends → SERP from a prompt
- [awesome-web-scraping](https://github.com/g4dr/awesome-web-scraping)
