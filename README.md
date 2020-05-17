## Introduction

Welcome to the TickerTick API!

The TickerTick API provides fresh and relevant stock news stories based on a powerful query language. The API covers all US stocks. The source websites of the news stories include a lot of well-known websites and all company filings.

An example web app using TickerTick API.
- [https://tickertick.com](https://tickertick.com/?q=FB%2CAMZN%2CAAPL%2CNFLX%2CGOOG&lang=en)

All requests to the TickerTick API are made via GET to `https://api.tickertick.com/`.

Below are the available endpoints:


## Get stock news feed API
This API returns a feed of the lastest news stories relevant to the query in reverse chronological order.

### Endpoint
`GET` https://api.tickertick.com/feed

### Example request URL
Stock news feed for Apple Inc. - https://api.tickertick.com/feed?q=f3:aapl&lang=en

[Rendered stock news feed for Apple Inc.](https://tickertick.netlify.com/?q=AAPL)

### Parameters

| Parameter  | Description                     | Options                            |  Example value    |
|----------------|-------------------------------|--------------------------------------------------------|------------|
| q              | The query string       | Any query string in a query language<br>(explained below) | `f3:aapl`  |
| lang           | The language of the returned news stories | en: English<br>cn: Chinese<br>en,cn: Both  | `en,cn` |
| n              | How many latest news stories to fetch|   Any number between 1 and 1000 |      `42`      |
| last    | A story id for pagination.<br>Fetch news stories older than the story with this id.| A 64 bit integer. Each returned news story has such an id. |  `6844326865886118959`       |

### The query language
The query language is a [Context free language](https://en.wikipedia.org/wiki/Context-free_language) following the grammar below
```
query => term | (and query_list) | (or query_list) | (diff query query)

query_list => query query_list | term

term => f_term:any_stock_ticker | tx:any_string | s:any_website_domain_name

f_term => f0 | f1 | f2 | f3

```
### Operator semantics

| operator | semantics|
|---------------|-----------|
| `(and query_list)` | request news stories matching all queries in `query_list`|
| `(or query_list)` | request news stories matching any query in `query_list`|
| `(diff query1 query2)` | request news stories matching `query1` but not `query2`|


### Term semantics
| term | semantics |
|---------------|-----------|
| `f0:stock_ticker`| Request company filings only about `stock_ticker`   |
| `f1:stock_ticker`| Request most relevant stories about `stock_ticker`  |
| `f2:stock_ticker`| Request most relevant and medium relevant stories about `stock_ticker`  |
| `f3:stock_ticker`| Request all stories about `stock_ticker`  |
| `tx:any_string`| request news story matching text `any_string`. <br> (`any_string` can have whitespaces)  |
| `s:domain_name`| request news story from websites on domain `domain_name` <br> (`domain_name` shouldn't contain '.' or '/')  |


### Example queries
| Example query | Semantics | URL |
|---------------|-----------|-------|
| `f0:aapl`  | Relevant level 0 news stories about Apple Inc. (ticker: aapl)| [https://api.tickertick.com/feed?q=f0:aapl](https://api.tickertick.com/feed?q=f0:aapl)      |
| `(or f3:fb f3:amzn f3:nflx f3:goog)` | Relevant level 3 news stories about [FANG stocks](https://www.investopedia.com/terms/f/fang-stocks-fb-amzn.asp) | https://api.tickertick.com/feed?q=(or%20f3:fb%20f3:amzn%20f3:nflx%20f3:goog)|
| `(diff tx:elon musk s:nytimes)` | Stories about Elon Musk not from NY times | https://api.tickertick.com/feed?q=(diff%20tx:elon%20musk%20s:nytimes)|

### The returned news stories
The returned news stories are in a JSON string. The example from URL https://api.tickertick.com/feed?q=f3:aapl&lang=en&n=2

```
{
  "stories": [
    {
      "id": "-6182621638785416990",
      "title": "‘The time is now to have a federal privacy bill,’ says Tim Cook",
      "url": "https://www.theverge.com/2019/11/22/20978140/tim-cook-apple-federal-privacy-bill-facebook-breakup-big-tech",
      "site": "www.theverge.com",
      "time": 1574547739000,
      "favicon_url": "https://cdn.vox-cdn.com/uploads/chorus_asset/file/7395351/android-chrome-192x192.0.png"
    },
    {
      "id": "-3255988935767362050",
      "title": "Email App Maker Begs Apple CEO to Get Back on the App Store Author: Julian Chokkattu Julian Chokkattu",
      "url": "https://www.wired.com/story/bluemail-developers-write-open-letter-to-apple-mac-app-store/",
      "site": "www.wired.com",
      "time": 1574547739000,
      "favicon_url": "https://www.wired.com/images/logos/w-icon-512x512.png"
    },
    {
      "id": "4112141090713067885",
      "title": "NYU professor Scott Galloway: Disney+ and Apple TV+ will be the winners of the streaming wars",
      "url": "https://www.businessinsider.com/nyu-professor-scott-galloway-disney-plus-apple-tv-streaming-wars-2019-11",
      "site": "www.businessinsider.com",
      "time": 1574547739000,
      "favicon_url": "http://static.businessinsider.de/assets/images/de/favicons/android-chrome-512x512.png"
    },
  ]
}

```
| Story field | Description |
|---------------|-----------|
| `id`  | A unique string id of the story. The `id` can be used for pagination as the value of parameter `last`.|
| `title` | The title of the news story in HTML format. |
| `url` | The url of the news story. |
| `site` | The source website of the news story. |
| `time` | The timestamp of the news story. The same semantics as `Date.now()` in Javascript. |
| `favicon_url` | The url of the favicon of the source website. |

### Top news source websites
| Website                      | #stories in recent 12 weeks |
|----------------------|---------------|
| sec.gov               |   148105 |
| wsj.com               |    16247 |
| businessinsider.com   |     9971 |
| globenewswire.com     |     7399 |
| businesswire.com      |     6882 |
| twitter.com               |     5884 |
| apnews.com            |     5366 |
| barrons.com           |     5217 |
| cnbc.com              |     4544 |
| nytimes.com           |     4117 |
| forbes.com            |     4011 |
| bloomberg.com         |     3635 |
| marketwatch.com       |     3198 |
| theverge.com          |     2968 |
| wired.com             |     2783 |
| techcrunch.com            |     2404 |
| prnewswire.com        |     2272 |
| nypost.com                |     2002 |
| fastcompany.com       |     1832 |
| vox.com               |     1491 |
| axios.com             |     1422 |

## Stock ticker search API

### Endpoint
`GET` https://api.tickertick.com/tickers


### Parameters
| Parameter  | Description                     | Options                            |  Example value    |
|----------------|-------------------------------|--------------------------------------------------------|------------|
| p              | The query string to match the company name or the stock ticker.      | Any string.| `Tesl`  |
| n            | How many tickers to return at most.     | Any integer.| `4`  |

### Example request URL
The returned result is a JSON string consisting of all matched stock tickers.

Search for any tickers matching `Ama` - https://api.tickertick.com/tickers?p=Ama&n=2
```
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
Search for any tickers matching `aa` - https://api.tickertick.com/tickers?p=aa&n=2
```
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
