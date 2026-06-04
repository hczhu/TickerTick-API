---
name: tickertick
---

Query the TickerTick API for stock news stories and ticker search. Base URL: `https://api.tickertick.com`. Rate limit: 10 req/min per IP (>5 in any 30s window → 30s block).

## Endpoints

### `/feed` — news stories
```
GET https://api.tickertick.com/feed?q=<query>&n=<1-200>&last=<story_id>
```
Returns stories in reverse-chronological order. `last` paginates using a story's `id`. Story `time` is epoch **milliseconds**.

### `/tickers` — ticker search
```
GET https://api.tickertick.com/tickers?p=<prefix>&n=<count>
```
Search by company name or ticker prefix. Use this to resolve a company name to a ticker before querying `/feed`.

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
| `E:entity` | Entity in title — lowercase, spaces→`_` (e.g. `E:elon_musk`, `E:nvidia`) |
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
Then find stories with `similar_stories` arrays of length >3 — those are the most-covered stories. Stories tagged `_best_` are editorially curated highlights.

**SEC filings for a ticker:**
```
https://api.tickertick.com/feed?q=(and tt:aapl T:sec_fin)&n=10
```

**Entity search (e.g. person, product):**
```
https://api.tickertick.com/feed?q=E:jensen_huang&n=20
```

**Source-specific:**
```
https://api.tickertick.com/feed?q=(and tt:tsla s:reuters)&n=20
```

## Response Fields

| Field | Description |
|---|---|
| `id` | Unique story ID; use as `last=` for pagination |
| `title` | Story headline |
| `url` | Story URL |
| `site` | Source domain |
| `time` | Publish time in epoch milliseconds |
| `tags` | Ticker tags (present when `tt:` term used) |
| `similar_stories` | IDs of similar stories in the same response (clustering signal) |
| `description` | Optional summary |

## Ticker Search Example

To find the ticker for "Micron Technology":
```
https://api.tickertick.com/tickers?p=Micron&n=5
```
