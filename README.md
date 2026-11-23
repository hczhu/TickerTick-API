# TickerTick API

The TickerTick API provides the latest stock news stories through a powerful query language. It covers all companies listed in US stock markets ([around 10,000 tickers](https://api.tickertick.com/tickers?n=9999)) and [hundreds of top startups](https://api.tickertick.com/tickers?n=9999&p=.), drawing news from around ten thousand source websites.

## Quick Start

> Goal: make your first successful API call in under a minute.

Fetch [the 200 latest news stories for ticker `aapl`](https://api.tickertick.com/feed?q=z:aapl&n=200) (Apple Inc.):

```
https://api.tickertick.com/feed?q=z:aapl&n=200
```

Replace `aapl` with any ticker and `200` with any number between 1 and 200. The response is a JSON array of stories in reverse chronological order (see [Response](#response) for the full schema).

The `q` (query) parameter is the heart of the API. The two simplest terms select news by ticker:

| Term | Meaning | Example |
|---|---|---|
| `z:TICKER` | News stories about a ticker | `z:aapl`, `z:tsla` |
| `tt:TICKER` | A broader set of news stories about a ticker (more than `z:`) | `tt:aapl`, `tt:tsla` |

That's enough to get started. Terms can be combined with `and` / `or` / `diff` operators and extended with several other term types — see the full [Query Language](#query-language) below.

### Quick Start with Claude Code

[Claude Code](https://claude.ai/code) is Anthropic's AI coding agent. Install the TickerTick skill to let Claude query stock news on your behalf using natural language.

**Recommended — install as a plugin.** This repo doubles as a Claude Code plugin marketplace, so you can pull later updates with `claude plugin marketplace update tickertick-api`:

```bash
claude plugin marketplace add hczhu/TickerTick-API
claude plugin install tickertick@tickertick-api
```

**Or install the skill file by hand.** A Claude Code skill is a directory under `~/.claude/skills/` containing a `SKILL.md`:

```bash
mkdir -p ~/.claude/skills/tickertick
curl -fsSL https://raw.githubusercontent.com/hczhu/TickerTick-API/master/skills/tickertick/SKILL.md \
  -o ~/.claude/skills/tickertick/SKILL.md
```

Already cloned the repo? `cp skills/tickertick/SKILL.md ~/.claude/skills/tickertick/SKILL.md` does the same.

Once installed, start a new Claude Code session and invoke it with `/tickertick`. Claude will translate your request into the appropriate query and fetch results — for example:

> *"Get the latest curated news for NVDA and AMD, excluding Reddit"*

> *"Fetch the most important stories about Apple this week"*

> *"Show me recent SEC filings for Microsoft"*

### Quick Start with Codex

[Codex](https://developers.openai.com/codex) is OpenAI's coding agent. This repo is also a Codex plugin marketplace, so the same skill installs there:

```bash
codex plugin marketplace add hczhu/TickerTick-API
codex plugin add tickertick@tickertick-api
```

Refresh later with `codex plugin marketplace upgrade`, and remove it with `codex plugin remove tickertick`.

To install the skill file by hand instead, Codex reads skills from `~/.codex/skills/<name>/SKILL.md`:

```bash
mkdir -p ~/.codex/skills/tickertick
curl -fsSL https://raw.githubusercontent.com/hczhu/TickerTick-API/master/skills/tickertick/SKILL.md \
  -o ~/.codex/skills/tickertick/SKILL.md
```

### Quick Start with PyTickerTick

[PyTickerTick](https://pypi.org/project/pytickertick/) is a Python wrapper for the TickerTick API:

```
pip install pytickertick
```

```python
import tickertick as tt
import tickertick.query as query

feed = tt.get_feed(
    query = query.And(
        query.BroadTicker('aapl'),
        query.StoryType(query.StoryTypes.SEC)
    )
)  # SEC filings from Apple Inc.
```

## Historical Data

> Goal: get bulk/past news data instead of the live feed.

If you are looking for historical data — e.g., all stock news stories from the past two years — check out [the releases page](https://github.com/hczhu/TickerTick-API/releases).

## About This Project

> Goal: understand what TickerTick is, how you may use it, and how it works.

Apps built on top of the TickerTick API:
- TickerTick web app: [https://tickertick.com](https://tickertick.com)
- An OpenAI custom GPT: [TickerTick GPT](https://chat.openai.com/g/g-MUCY7WB5O-tickertick-gpt)

### Terms of Use
- The API is free of commercial/non-commercial use.
- All endpoints have a rate limit of __10 requests per minute__ from the same IP address. The service enforces this. More precisely, an IP will be blocked for 30 seconds if more than 5 requests are sent within any 30 second time window.
- You are welcome to file an issue if you see any problem, like irrelevant stories.

> Email me if you need a higher request rate.

### API Use Cases
- **[thestockmarketwatch.com](https://thestockmarketwatch.com)** — calls the API about 10,000 times per day to get the latest news stories for stocks.
- **[TickerTick.com](https://tickertick.com)** — provides the broadest stock news.
- **[Trading Saga](https://tradingsaga.com/)** — a quick and immersive trading game that uses the API to fetch stock news.

### Acknowledgments
- **[spaCy](https://github.com/explosion/spaCy)** — the backend uses the English Transformer pipeline [en_core_web_trf](https://spacy.io/models/en#en_core_web_trf) to extract named entities from text and accomplish other NLP tasks.
- **[MediaWiki API](https://www.wikidata.org/w/api.php)** — used to get entities related to a ticker, e.g. `Elon Musk` for `TSLA` and `WhatsApp` for `FB`.
- **[Favicon Grabber](https://github.com/antongunov/favicongrabber.com)** — used to fetch website favicons.

### Contact
Mark(Hongcheng) Zhu - [support AT tickertick.com](mailto:support@tickertick.com)

## Query Language

> Goal: build complex queries by combining operators and terms.

The query language is a [context-free language](https://en.wikipedia.org/wiki/Context-free_language) following the grammar below:

```
query --> term | (and query_list) | (or query_list) | (diff query query)

query_list --> query query_list | term

term --> z:any_stock_ticker | tt:any_stock_ticker | TT:any_stock_ticker | s:any_website_domain_name | T:story_type | tag:_best_
```

### Operators

| Operator | Semantics |
|---|---|
| `(and query_list)` | News stories matching all queries in `query_list` |
| `(or query_list)` | News stories matching any query in `query_list` |
| `(diff query1 query2)` | News stories matching `query1` but not `query2` |

### Terms

| Term | Semantics | Examples |
|---|---|---|
| `z:stock_ticker` | News stories about `stock_ticker`. | `z:aapl` <br> `z:tsla` |
| `tt:stock_ticker` | More broad news stories about `stock_ticker` (more than `z:stock_ticker`). | `tt:aapl` <br> `tt:tsla` |
| `TT:stock_ticker` | The broadest set of news stories about `stock_ticker` (more than `tt:stock_ticker`). | `TT:aapl` <br> `TT:tsla` |
| `s:domain_name` | News stories from websites on domain `domain_name` (`domain_name` shouldn't contain '.' or '/'). | `s:wsj` <br> `s:cnbc` |
| `T:story_type` | News stories of a specific type. See [Story Types](#story-types). | `T:curated` <br> `T:sec` |
| `tag:_best_` | Editorially curated highlights. Matching stories carry `_best_` as an entry in their `tags` array. See [Retrieving the Most Important News](#retrieving-the-most-important-news). | `tag:_best_` |

### Story Types

| Term | Story type | Example query |
|---|---|---|
| T:curated | News stories from a curated list of [top financial/technology news sources](https://github.com/hczhu/TickerTick-API/blob/master/docs/top-news-sources.txt) | [T:curated](https://api.tickertick.com/feed?q=T:curated) |
| T:earning | Company earnings news (e.g. presentations, transcripts) | [T:earning](https://api.tickertick.com/feed?q=T:earning) |
| T:market | Stock market news | [T:market](https://api.tickertick.com/feed?q=T:market) |
| T:sec | SEC filings | [T:sec](https://api.tickertick.com/feed?q=T:sec) |
| T:sec_fin | Quarterly/annual financial reports | [T:sec_fin](https://api.tickertick.com/feed?q=T:sec_fin) |
| T:trade | Trading news | [T:trade](https://api.tickertick.com/feed?q=T:trade) |
| T:ugc | News stories from a curated list of [user-generated content platforms](https://github.com/hczhu/TickerTick-API/blob/master/docs/ugc-sources.txt), e.g. Reddit. | [T:ugc](https://api.tickertick.com/feed?q=T:ugc) |
| T:analysis | Stock analysis articles from [a curated list of sources](https://github.com/hczhu/TickerTick-API/blob/master/docs/top-news-sources.txt) | [T:analysis](https://api.tickertick.com/feed?q=T:analysis) |
| T:industry | Industry publications from [a curated list of sources](https://github.com/hczhu/TickerTick-API/blob/master/docs/industry-publications.txt) | [T:industry](https://api.tickertick.com/feed?q=T:industry) |
| T:fin_news | Financial news, from a broader set of sources than `T:curated` | [T:fin_news](https://api.tickertick.com/feed?q=T:fin_news) |

### Example Queries

| Example query | Semantics | API call URL |
|---|---|---|
| `(and tt:aapl s:sec)` | SEC filings from Apple Inc. (ticker: aapl) | [https://api.tickertick.com/feed?q=(and tt:aapl s:sec)](https://api.tickertick.com/feed?q=(and%20tt:aapl%20s:sec)) |
| `(or tt:meta tt:aapl tt:amzn tt:nflx tt:goog)` | News stories about [FAANG stocks](https://www.investopedia.com/terms/f/faang-stocks.asp) | [https://api.tickertick.com/feed?q=(or tt:meta tt:aapl tt:amzn tt:nflx tt:goog)](https://api.tickertick.com/feed?q=(or%20tt:meta%20tt:aapl%20tt:amzn%20tt:nflx%20tt:goog)) |
| `(and (or tt:meta tt:goog) s:reddit)` | News stories about Meta (`meta`) and Google (`goog`) from `reddit.com` | [https://api.tickertick.com/feed?q=(and (or tt:meta tt:goog) s:reddit)](https://api.tickertick.com/feed?q=(and%20(or%20tt:meta%20tt:goog)%20s:reddit)) |
| `(diff (or tt:meta tt:goog) s:reddit)` | News stories about Meta (`meta`) and Google (`goog`) not from Reddit.com | [https://api.tickertick.com/feed?q=(diff (or tt:meta tt:goog) s:reddit)](https://api.tickertick.com/feed?q=(diff%20(or%20tt:meta%20tt:goog)%20s:reddit)) |

### Retrieving the Most Important News

Story importance is derived from how widely a story is covered. The running example below retrieves the most important news for `$META` and `$AMZN`.

1. Retrieve candidate stories with [this query](https://api.tickertick.com/feed?n=200&q=(diff%20(or%20(or%20(or%20TT:amzn%20TT:meta)%20(and%20tag:_best_%20(or%20tt:meta%20tt:amzn)%20(or%20T:fin_news%20T:analysis%20T:industry))))%20(or%20T:trade%20T:market%20T:ugc))), which uses the broad `TT:` terms and the `tag:_best_` term while excluding trading, market and user-generated noise:

```
(diff (or (or (or TT:meta TT:amzn)
       (and tag:_best_ (or tt:meta tt:amzn) (or T:fin_news T:analysis T:industry))))
 (or T:trade T:market T:ugc))
```

2. Look for stories in the response that have a `similar_stories` array. A story together with its similar stories forms a **cluster**, and the story carrying the array is the cluster's **primary story**.
3. If a cluster holds more than 3 stories (tunable), its primary story is considered most important.
4. If any story in the cluster has `_best_` in its `tags`, that story is considered most important.

The story below satisfies both criteria:

```json
{
  "title": "Amazon plans to start delivering packages by drone in Texas later this year",
  "tags": [
    "amzn",
    "_best_"
  ],
  "similar_stories": [
    "7861538917579425004",
    "1801298483307348605",
    "-6273779958355143798",
    "1915490391207104498",
    "7429249118825970200"
  ]
}
```

## API Reference

> Goal: full endpoint, parameter, and response details.

### `GET` https://api.tickertick.com/feed

Returns a feed of the latest news stories relevant to the query, in reverse chronological order.

Example: news stories about Apple Inc. (ticker `aapl`) — https://api.tickertick.com/feed?q=tt:aapl

#### Parameters

| Parameter | Description | Options | Example value |
|---|---|---|---|
| q | The query string | Any string in the [Query Language](#query-language) | `(or tt:aapl tt:amzn)` |
| n | How many news stories to fetch | Any number between 1 and 200 | `42` |
| last | A story id for pagination. Fetch news stories older than the story with this id. | A 64-bit integer. Each returned news story has an id. | `6844326865886118959` |

#### Example API call (pagination)

| URL parameters | Semantics | API call URL |
|---|---|---|
| `q=tt:amzn`<br>`last=1866158884274957563`<br>`n=5` | Get 5 stories about Amazon (`amzn`) older than story with id `1866158884274957563` | https://api.tickertick.com/feed?q=tt:amzn&last=1866158884274957563&n=5 |

#### Response

The response is a JSON blob consisting of an array of stories in reverse chronological order. Each story has the following fields:

| Story field | Description |
|---|---|
| `id` | A unique string id of the story. Can be used for pagination as the value of the parameter `last`. |
| `title` | The title of the news story. |
| `url` | The url of the news story. |
| `site` | The source website of the news story. |
| `time` | The timestamp of the news story, in milliseconds since the Unix epoch (1970-01-01T00:00:00Z UTC) — same semantics as `Date.now()` in JavaScript. |
| `favicon_url` | The url of the favicon of the source website. |
| `tags` | An array of strings. Each string is the ticker for which the story is. Present only when a `tt:` term is in the query. |
| `similar_stories` | An array of story ID strings referencing other stories in the response that are considered similar to this one. Optional. |
| `description` | The description of the news story. Optional. |

<!---
> :warning: It's a known issue that some story timestamps are not accurate to varying degrees depending on the source websites. However, the overall accuracy should be fairly good. Most of story timestamps should be at least within one day of real story publication times. The root cause of this issue is a trade-off between backend resource usage and accuracy.
>
-->

An example response from request URL [https://api.tickertick.com/feed?q=(and T:curated tt:aapl)&n=30](https://api.tickertick.com/feed?q=(and%20T:curated%20tt:aapl)&n=30):

```json
{
  "stories": [
    {
      "id": "426800427666858718",
      "title": "Apple Reality Pro VR Headset: New Leak Reveals Unprecedented Detail",
      "url": "https://www.forbes.com/sites/davidphelan/2023/06/03/apple-reality-pro-vr-headset-new-leak-reveals-unprecedented-detail/",
      "site": "forbes.com",
      "time": 1685790913000,
      "favicon_url": "https://static.tickertick.com/website_icons/forbes.com.ico",
      "tags": [
        "aapl"
      ],
      "similar_stories": [
        "-3592774926732722485",
        "5969505199123739307",
        "-4590978801798836365"
      ],
      "description": "It’s a new mixed-reality headset and it’s a very big deal. It’s a new mixed-reality headset and it’s a very big deal.",
      "tickers": [
        "aapl"
      ]
    },
    ...
    {
      "id": "7335329458973640129",
      "title": "Apple’s Rumored VR Headset Has Sent Its Rivals Scrambling",
      "url": "https://www.wired.com/story/apple-vr-headset-sent-rivals-scrambling/",
      "site": "wired.com",
      "time": 1685790000000,
      "favicon_url": "https://static.tickertick.com/website_icons/wired.com.ico",
      "tags": [
        "aapl"
      ],
      "description": "If Apple announces a highly anticipated VR headset at its upcoming Worldwide Developers Conference, it may validate work by other companies in the industry.",
      "tickers": [
        "aapl"
      ]
    },
    {
      "id": "-560799274878500337",
      "title": "PayPal: Don't Fear Apple Pay And Stripe | $AAPL $AXP $MA $V $PYPL",
      "url": "https://seekingalpha.com/article/4609213-paypal-dont-fear-apple-pay-stripe",
      "site": "seekingalpha.com",
      "time": 1685787446000,
      "favicon_url": "https://static.tickertick.com/website_icons/seekingalpha.com.ico",
      "tags": [
        "aapl"
      ],
      "tickers": [
        "aapl"
      ]
    }
  ],
  "last_id": "-560799274878500337"
}
```

### `GET` https://api.tickertick.com/tickers

Search for stock tickers by company name or ticker prefix.

#### Parameters

| Parameter | Description | Options | Example value |
|---|---|---|---|
| p | The query string to match the company name or the stock ticker. | Any string. | `Tesl` |
| n | How many tickers to return at most. | Any integer. | `4` |

#### Response

The returned result is a JSON string consisting of all matched stock tickers.

Search for any tickers matching `Ama` — https://api.tickertick.com/tickers?p=Ama&n=2

```json
{
  "tickers": [
    {
      "ticker": "amzn",
      "company_name": "Amazon.com, Inc."
    },
    {
      "ticker": "amag",
      "company_name": "AMAG Pharmaceuticals, Inc."
    }
  ]
}
```

Search for any tickers matching `aa` — https://api.tickertick.com/tickers?p=aa&n=2

```json
{
  "tickers": [
    {
      "ticker": "aapl",
      "company_name": "Apple Inc."
    },
    {
      "ticker": "aal",
      "company_name": "American Airlines Group, Inc."
    }
  ]
}
```
