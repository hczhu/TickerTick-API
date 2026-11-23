---
name: tickertick
description: Query the TickerTick API for stock news stories and ticker search. Use when the user asks for news about a public company or ticker, SEC filings, earnings coverage, analyst or industry articles, or wants to resolve a company name to a ticker symbol.
---

Query the TickerTick API for stock news stories and ticker search. Base URL: `https://api.tickertick.com`. Rate limit: 10 req/min per IP (>5 in any 30s window → 30s block).

## Endpoints

### `/feed` — news stories
```
GET https://api.tickertick.com/feed?q=<query>&n=<1-200>&last=<story_id>
```
Returns `{"stories": [...], "last_id": "..."}` — stories in reverse-chronological order. Story `time` is epoch **milliseconds**.

To paginate, pass the response's `last_id` (or any story's `id`) back as `last=`. Story ids are **quoted strings** and may be negative, e.g. `"-8901745850278641194"`; they are 64-bit values, so never parse them as JavaScript numbers.

### `/tickers` — ticker search
```
GET https://api.tickertick.com/tickers?p=<prefix>&n=<count>
```
Search by company name or ticker prefix. Use this to resolve a company name to a ticker before querying `/feed`. Returns:

```json
{"tickers": [{"ticker": "amzn", "company_name": "Amazon.com, Inc."}]}
```

## Query Language

```
query  →  term
       |  (and query_list)
       |  (or  query_list)
       |  (diff query1 query2)    # query1 minus query2
```

### Terms

| Term | Meaning |
|---|---|
| `z:TICKER` | Narrow: stories directly about the ticker |
| `tt:TICKER` | Broad: wider set of stories about the ticker |
| `TT:TICKER` | Broadest: even more stories (use for important-news queries) |
| `s:DOMAIN` | Source domain — no dots or slashes (e.g. `s:wsj`, `s:cnbc`, `s:reddit`) |
| `T:TYPE` | Story type — see table below |
| `tag:_best_` | Editorially curated "best" stories (use with `TT:` for important news) |

### Story Types (`T:`)

| Type | Meaning |
|---|---|
| `T:curated` | Top financial/tech news sources |
| `T:analysis` | Stock analysis articles |
| `T:industry` | Industry publications |
| `T:earning` | Earnings presentations and transcripts |
| `T:sec` | SEC filings |
| `T:sec_fin` | Quarterly/annual financial reports |
| `T:fin_news` | Financial news (broader than curated) |
| `T:market` | Stock market news |
| `T:trade` | Trading news |
| `T:ugc` | User-generated content (Reddit, forums) |

## Common Patterns

**Latest news for a ticker:**
```
https://api.tickertick.com/feed?q=tt:nvda&n=20
```

**Curated news only:**
```
https://api.tickertick.com/feed?q=(and tt:nvda T:curated)&n=20
```

**Multiple tickers:**
```
https://api.tickertick.com/feed?q=(or tt:nvda tt:amd tt:intc)&n=30
```

**Exclude Reddit/UGC noise:**
```
https://api.tickertick.com/feed?q=(diff tt:nvda T:ugc)&n=20
```

**Most important news (cluster-based):**
```
(diff
  (or (or (or TT:TICKER1 TT:TICKER2)
          (and tag:_best_ (or tt:TICKER1 tt:TICKER2) (or T:fin_news T:analysis T:industry))))
  (or T:trade T:market T:ugc))
```
A story plus its `similar_stories` forms a cluster, and the story carrying them is the cluster's primary story. A cluster with more than 3 stories makes its primary story important. If any story in the cluster has `_best_` in its `tags`, that story wins — `_best_` is delivered as an entry in `tags` alongside the tickers, not as a separate field.

**SEC filings for a ticker:**
```
https://api.tickertick.com/feed?q=(and tt:aapl T:sec_fin)&n=10
```

**Source-specific:**
```
https://api.tickertick.com/feed?q=(and tt:tsla s:reuters)&n=20
```

## Response Fields

| Field | Description |
|---|---|
| `id` | Unique story id, a string; use as `last=` for pagination |
| `title` | Story headline |
| `url` | Story URL |
| `site` | Source website |
| `time` | Publish time in epoch milliseconds (as `Date.now()`) |
| `favicon_url` | Favicon of the source website |
| `tags` | Tickers the story is about, present when a `tt:` term is in the query. May also contain `_best_` |
| `tickers` | Duplicate of `tags`, same contents including `_best_`. Undocumented in the README; prefer `tags` |
| `similar_stories` | Ids of similar stories in the same response (clustering signal). Optional |
| `description` | Optional summary |

The enclosing object also carries `last_id`, the id of the oldest story returned — pass it as `last=` to fetch the next page.

## Ticker Search Example

To find the ticker for "Micron Technology":
```
https://api.tickertick.com/tickers?p=Micron&n=5
```
