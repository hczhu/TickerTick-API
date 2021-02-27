## Introduction

Welcome to the TickerTick API!

The TickerTick API provides fresh and relevant stock news stories based on a powerful query language. The API covers all companies listed in US stock markets. The source websites of the news stories include a lot of well-known websites and all company filings.

Take a look at an example web app that uses TickerTick API.
- [https://tickertick.com](https://tickertick.com/?q=FB%2CAMZN%2CAAPL%2CNFLX%2CGOOG&lang=en)


### Terms of Use
- The API is free of commercial/non-commercialuse use.
- All endpoints have a rate limit of __6 requests per minute__ from the same IP address. This is enforced by the service.
- You are welcome to file an issue if you see any problem, like irrelevant stories.

## Get stock news feed API
This API returns a feed of the lastest news stories relevant to the query in reverse chronological order.


### Endpoint
`GET` https://api.tickertick.com/feed

### Example request URL
News stories about Apple Inc. - https://api.tickertick.com/feed?q=tt:aapl&lang=en

[Rendered stock news feed for Apple Inc.](https://tickertick.com/?q=AAPL)

### Parameters

| Parameter  | Description                     | Options                            |  Example value    |
|----------------|-------------------------------|--------------------------------------------------------|------------|
| q              | The query string       | Any query string in a query language<br>(explained below) | `tt:aapl`  |
| lang           | Comma-separated languages of requested feed stories| `en`: English<br>`cn`: Chinese<br>`en,cn`: Both  | `en,cn` |
| n              | How many latest news stories to fetch|   Any number between 1 and 1000 |      `42`      |
| last    | A story id for pagination.<br>Fetch news stories older than the story with this id.| A 64 bit integer. Each returned news story has such an id. |  `6844326865886118959`       |
| hours_ago | Only return stories from this number of hours ago.| A 32 bit positive integer. | `2400` (100 days ago) <br> `24000` (1000 days ago)|
| excl  | Comma-separated types of stories to exclude from the requested feed  | `filings`: exclude  company filings<br> `ugc`: exclude user generated content | `filings,ugc` |


### The query language
The query language is a [Context free language](https://en.wikipedia.org/wiki/Context-free_language) following the grammar below
```
query => term | (and query_list) | (or query_list) | (diff query query)

query_list => query query_list | term

term => tt:any_stock_ticker | tx:any_string | s:any_website_domain_name


```
### Operator semantics

| operator | semantics|
|---------------|-----------|
| `(and query_list)` | Request news stories matching all queries in `query_list`|
| `(or query_list)` | Request news stories matching any query in `query_list`|
| `(diff query1 query2)` | Request news stories matching `query1` but not `query2`|


### Term semantics
| term | semantics |
|---------------|-----------|
| `tt:stock_ticker`| Request news stories about `stock_ticker`  |
| `tx:any_string`| Request news stories with titles matching text `any_string`. <br> (`any_string` can have whitespaces)  |
| `s:domain_name`| Request news stories from websites on domain `domain_name` <br> (`domain_name` shouldn't contain '.' or '/')  |


### Example queries
| Example query | Semantics | API call URL | Rendered stories |
|---------------|-----------|-------|---------------------------------|
| `(and tt:aapl s:sec)`  | SEC filings from Apple Inc. (ticker: aapl)| [https://api.tickertick.com/feed?q=(and tt:aapl s:sec)](https://api.tickertick.com/feed?q=(and%20tt:aapl%20s:sec))      | [`(and tt:aapl s:sec)`](https://s.tickertick.com/search.html?q=(and%20tt:aapl%20s:sec)) |
| `(or tt:fb tt:amzn tt:nflx tt:goog)` | News stories about [FANG stocks](https://www.investopedia.com/terms/f/fang-stocks-fb-amzn.asp) | [https://api.tickertick.com/feed?q=(or tt:fb tt:amzn tt:nflx tt:goog)](https://api.tickertick.com/feed?q=(or%20tt:fb%20tt:amzn%20tt:nflx%20tt:goog)) | [`(or tt:fb tt:amzn tt:nflx tt:goog)`](https://s.tickertick.com/search.html?q=(or%20tt:fb%20tt:amzn%20tt:nflx%20tt:goog)) |
| `(and (or tt:fb tt:goog) s:reddit)` | News stories about Facebook (`fb`) and Google (`goog`) from `reddit.com` | [https://api.tickertick.com/feed?q=(and (or tt:fb tt:goog) s:reddit)](https://api.tickertick.com/feed?q=(and%20(or%20tt:fb%20tt:goog)%20s:reddit))|[`(and (or tt:fb tt:goog) s:reddit)`](https://s.tickertick.com/search.html?q=(and%20(or%20tt:fb%20tt:goog)%20s:reddit)) |
| `(diff (or tt:fb tt:goog) s:reddit)` | News stories about Facebook (`fb`) and Google (`goog`) not from Reddit.com | [https://api.tickertick.com/feed?q=(diff (or tt:fb tt:goog) s:reddit)](https://api.tickertick.com/feed?q=(diff%20(or%20tt:fb%20tt:goog)%20s:reddit))|[`(diff (or tt:fb tt:goog) s:reddit)`](https://s.tickertick.com/search.html?q=(diff%20(or%20tt:fb%20tt:goog)%20s:reddit)) |
| `(diff tx:elon musk s:nytimes)` | Stories with `Elon Musk` in titles not from NY times | [https://api.tickertick.com/feed?q=(diff tx:elon musk s:nytimes)](https://api.tickertick.com/feed?q=(diff%20tx:elon%20musk%20s:nytimes))| [`(diff tx:elon musk s:nytimes)`](https://s.tickertick.com/search.html?q=(diff%20tx:elon%20musk%20s:nytimes)) |

### Example API calls

| URL parameters | Semanrics | API call URL | Rendered stories |
|---------------|-----------|-------|---------------------------------|
| `q=tt:fb`<br>`hours_ago=14400`<br>`lang=en`<br>`n=5` | Get 5 news stories in English about Facebook(`fb`) from 600 days ago | https://api.tickertick.com/feed?q=tt:fb&hours_ago=14400&lang=en&n=5  | [`q=tt:fb&hours_ago=14400&lang=en&n=5`](https://s.tickertick.com/search.html?q=tt:fb&hours_ago=14400&lang=en&n=5) |
| `q=tt:fb`<br>`last=-8385103983945365342`<br>`n=5` | Get 5 stories about Facebook(`fb`) older than story with id `-8385103983945365342`| https://api.tickertick.com/feed?q=tt:fb&last=-8385103983945365342&n=5  | [`q=tt:fb&last=-8385103983945365342&n=5`](https://s.tickertick.com/search.html?q=tt:fb&last=-8385103983945365342&n=5) |


### The response
The response is a json blob consisting of an array of stories in reverse chronological order. Each story has the following fields

| Story field | Description |
|---------------|-----------|
| `id`  | A unique string id of the story. The `id` can be used for pagination as the value of parameter `last`.|
| `title` | The title of the news story in HTML format. |
| `url` | The url of the news story. |
| `site` | The source website of the news story. |
| `time` | The timestamp of the news story. It's the number of milliseconds since the "Unix epoch", 1970-01-01T00:00:00Z (UTC). The same semantics as `Date.now()` in Javascript. |
| `favicon_url` | The url of the favicon of the source website. |
| `tags` | An array of strings. Each string is the ticker the story is about. |

An example response from reqeust URL https://api.tickertick.com/feed?q=(or%20tt:fb%20tt:aapl)&lang=en&n=2

```
{
  "stories": [
    {
      "id": "-8234495239863619632",
      "title": "Apple Inc.: Form FWP - 2020-08-13",
      "url": "https://www.sec.gov/Archives/edgar/data/320193/0001193125-20-219289-index.html",
      "site": "www.sec.gov",
      "time": 1597302000000,
      "favicon_url": "https://www.sec.gov/themes/custom/secgov/favicon.ico",
      "tags": [
        "aapl"
      ]
    },
    {
      "id": "3888876773322702496",
      "title": "Facebook Inc: Form 4 - 2020-08-13",
      "url": "https://www.sec.gov/Archives/edgar/data/1326801/0000950103-20-015792-index.html",
      "site": "www.sec.gov",
      "time": 1597302000000,
      "favicon_url": "https://www.sec.gov/themes/custom/secgov/favicon.ico",
      "tags": [
        "fb"
      ]
    }
  ],
  "last_id": "3888876773322702496"
}

```


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
