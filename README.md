## Introduction

Welcome to the TickerTick API!

The TickerTick API provides fresh and relevant stock news stories through a powerful query language. The API covers all companies listed in US stock markets. The source websites of the news stories include around ten thousand websites.

Take a look at an example web app that uses TickerTick API.
- [https://tickertick.com](https://tickertick.com/?q=FB%2CAMZN%2CAAPL%2CNFLX%2CGOOG&lang=en)


### Terms of Use
- The API is free of commercial/non-commercialuse use.
- All endpoints have a rate limit of __6 requests per minute__ from the same IP address. This is enforced by the service.
- You are welcome to file an issue if you see any problem, like irrelevant stories.

> :warning: An IP address has a rate limit of __6 requests per minute__.

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
| lang           | Comma-separated languages of requested feed stories| `en`: English<br>`zh`: Chinese<br>`en,zh`: Both  | `en,zh` |
| n              | How many latest news stories to fetch|   Any number between 1 and 1000 |      `42`      |
| last    | A story id for pagination.<br>Fetch news stories older than the story with this id.| A 64 bit integer. Each returned news story has such an id. |  `6844326865886118959`       |
| hours_ago | Only return stories from this number of hours ago.| A 32 bit positive integer. | `2400` (100 days ago) <br> `24000` (1000 days ago)|
| excl  | Comma-separated types of stories to exclude from the requested feed  | `filings`: exclude  company filings<br> `ugc`: exclude user generated content (e.g. forum posts) | `filings,ugc` |


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
| `similar_stories` | An array of strings. Each string is a story id referencing another story in the response. The referenced stories are considered stories similar to this one.|

<!---
> :warning: It's a known issue that some story timestamps are not accurate to varying degrees depending on the source websites. However, the overall accuracy should be fairly good. Most of story timestamps should be at least within one day of real story publication times. The root cause of this issue is a trade-off between backend resource usage and accuracy.
> 
-->

An example response from reqeust URL https://api.tickertick.com/feed?q=(and%20l:en%20tt:fb%20t:533%20t:data)&n=44

```
{
  "stories": [
    ...
    {
      "id": "-8337688391260457343",
      "title": "533 million Facebook users’ personal data leaked online",
      "url": "https://www.cyberscoop.com/533-million-facebook-users-leaked-online-fraud-cybercrime/",
      "site": "cyberscoop.com",
      "time": 1617637314000,
      "favicon_url": "https://tickertick.com/website_icons/cyberscoop.com.ico",
      "tags": [
        "fb"
      ]
    },
    ...
    {
      "id": "6696963651995134378",
      "title": "Facebook data on 533 million users reemerge online for free",
      "url": "https://www.latimes.com/business/story/2021-04-03/facebook-data-hack",
      "site": "latimes.com",
      "time": 1617408000000,
      "favicon_url": "https://tickertick.com/website_icons/latimes.com.ico",
      "tags": [
        "fb"
      ],
      "similar_stories": [
        "-8337688391260457343",
        "-3821801551305481556",
        "-6524004435438445443",
        "2801015657891610459",
        "3614998498811631004",
        "-3992463990792441647",
        "1565025985713259443",
        "8800013191505707483",
        "-5314230086780537240",
        "8481455956967105134",
        "9138167294273170557",
        "-2370472546134774787",
        "-1825784017164803279"
      ]
    },
    ...
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
