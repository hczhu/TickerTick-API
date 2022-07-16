## Introduction

Welcome to the TickerTick API!

The TickerTick API provides fresh and relevant stock news stories through a powerful query language. The API covers all companies listed in US stock markets. The source websites of the news stories include around ten thousand websites.

Take a look at an example web app that uses TickerTick API.
- [https://tickertick.com](https://tickertick.com/?q=FB%2CAMZN%2CAAPL%2CNFLX%2CGOOG)


### Terms of Use
- The API is free of commercial/non-commercialuse use.
- All endpoints have a rate limit of __30 requests per minute__ from the same IP address. This is enforced by the service. More precisely, an IP will be blocked for one minute if more than 5 requests are sent within any 10 second time window.
- You are welcome to file an issue if you see any problem, like irrelevant stories.

> :warning: An IP address has a rate limit of __30 requests per minute__. Contact `mail AT tickertick.com` if you need a higher request rate.

## Get stock news feed API
This API returns a feed of the lastest news stories relevant to the query in reverse chronological order.


### Endpoint
`GET` https://api.tickertick.com/feed

### Example request URL
News stories about Apple Inc. (its ticker is __aapl__) - https://api.tickertick.com/feed?q=tt:aapl

[Rendered stock news feed for Apple Inc.](https://tickertick.com/ticker/aapl/feed)

### Parameters

| Parameter  | Description                     | Options                            |  Example value    |
|----------------|-------------------------------|--------------------------------------------------------|------------|
| q              | The query string       | Any query string in a query language<br>(explained below) | `(or tt:aapl tt:amzn)`  |
| n              | How many latest news stories to fetch|   Any number between 1 and 1000 |      `42`      |
| last    | A story id for pagination.<br>Fetch news stories older than the story with this id.| A 64 bit integer. Each returned news story has such an id. |  `6844326865886118959`       |
| hours_ago | Only return stories from this number of hours ago.| A 32 bit positive integer. | `2400` (100 days ago) <br> `24000` (1000 days ago)|


### The query language
The query language is a [Context-free Language](https://en.wikipedia.org/wiki/Context-free_language) following the grammar below
```
query --> term | (and query_list) | (or query_list) | (diff query query)

query_list --> query query_list | term

term --> tt:any_stock_ticker | s:any_website_domain_name | E:any_entity | T:story_type 

```
### Operator semantics

| Operator | Semantics|
|---------------|-----------|
| `(and query_list)` | Request news stories matching all queries in `query_list`|
| `(or query_list)` | Request news stories matching any query in `query_list`|
| `(diff query1 query2)` | Request news stories matching `query1` but not `query2`|


### Term semantics
| Term | Semantics | Examples |
|---------------|-----------|---|
| `tt:stock_ticker`| Request news stories about `stock_ticker`  | `tt:aapl`  <br>`tt:tsla` <br> `tt:COIN` |
| `TT:stock_ticker`| Request news stories about `stock_ticker` and only match entities in the story title. | `TT:aapl`  <br>`TT:tsla` <br> `TT:COIN` |
| `s:domain_name`| Request news stories from websites on domain `domain_name` <br> (`domain_name` shouldn't contain '.' or '/')  | `s:wsj` <br> `s:cnbc`|
| `E:any_entity`| Request news stories with titles semantically matching `any_entity`. <br> (replace any whitespace in `any_entity` by `_` )<br> (`any_entity` shold be in lower case)  | `E:shiba_inu`  <br> `E:rent_the_runway` <br> `E:elon_musk`  <br> `E:zoom` |
| `T:story_type`| Request news stories of a specific type. See [here](https://github.com/hczhu/TickerTick-API/edit/master/README.md#story-types) for the list of story types.  | `T:fin_news`  <br> `T:sec` |

### Story types
| Term | Story type| Example query |
|---------------|-----------|----|
| T:fin_news | News stories from a select set of [top financial/technology news sources](https://github.com/hczhu/TickerTick-API/blob/master/top-news-sources.txt) | [T:fin_news](https://api.tickertick.com/feed?q=T:fin_news)
| T:earning | Company earnings news (e.g. presentations, transcripts) | [T:earning](https://api.tickertick.com/feed?q=T:earning) 
| T:market | Stock market news | [T:market](https://api.tickertick.com/feed?q=T:market)
| T:sec | SEC filings | [T:sec](https://api.tickertick.com/feed?q=T:sec)
| T:sec_fin | Quarterly/annual financial reports | [T:sec_fin](https://api.tickertick.com/feed?q=T:sec_fin)
| T:trade | Trading news | [T:trade](https://api.tickertick.com/feed?q=T:trade)
| T:ugc | News stories from [user-generated content platforms](https://github.com/hczhu/TickerTick-API/blob/master/ugc-domains.txt), e.g. Reddit.  | [T:ugc](https://api.tickertick.com/feed?q=T:ugc)
| T:analysis | Stock analsis articles from [a curated list of soureces]() | [T:analysis](https://api.tickertick.com/feed?q=T:analysis)



### Example queries
| Example query | Semantics | API call URL | Rendered stories |
|---------------|-----------|-------|---------------------------------|
| `(and tt:aapl s:sec)`  | SEC filings from Apple Inc. (ticker: aapl)| [https://api.tickertick.com/feed?q=(and tt:aapl s:sec)](https://api.tickertick.com/feed?q=(and%20tt:aapl%20s:sec))      | [`(and tt:aapl s:sec)`](https://api.tickertick.com/search.html?q=(and%20tt:aapl%20s:sec)) |
| `(or tt:fb tt:amzn tt:nflx tt:goog)` | News stories about [FANG stocks](https://www.investopedia.com/terms/f/fang-stocks-fb-amzn.asp) | [https://api.tickertick.com/feed?q=(or tt:fb tt:amzn tt:nflx tt:goog)](https://api.tickertick.com/feed?q=(or%20tt:fb%20tt:amzn%20tt:nflx%20tt:goog)) | [`(or tt:fb tt:amzn tt:nflx tt:goog)`](https://api.tickertick.com/search.html?q=(or%20tt:fb%20tt:amzn%20tt:nflx%20tt:goog)) |
| `(and (or tt:fb tt:goog) s:reddit)` | News stories about Facebook (`fb`) and Google (`goog`) from `reddit.com` | [https://api.tickertick.com/feed?q=(and (or tt:fb tt:goog) s:reddit)](https://api.tickertick.com/feed?q=(and%20(or%20tt:fb%20tt:goog)%20s:reddit))|[`(and (or tt:fb tt:goog) s:reddit)`](https://api.tickertick.com/search.html?q=(and%20(or%20tt:fb%20tt:goog)%20s:reddit)) |
| `(diff (or tt:fb tt:goog) s:reddit)` | News stories about Facebook (`fb`) and Google (`goog`) not from Reddit.com | [https://api.tickertick.com/feed?q=(diff (or tt:fb tt:goog) s:reddit)](https://api.tickertick.com/feed?q=(diff%20(or%20tt:fb%20tt:goog)%20s:reddit))|[`(diff (or tt:fb tt:goog) s:reddit)`](https://api.tickertick.com/search.html?q=(diff%20(or%20tt:fb%20tt:goog)%20s:reddit)) |
| `(diff E:elon_musk s:nytimes)` | Stories with `Elon Musk` in titles not from NY times | [https://api.tickertick.com/feed?q=(diff E:elon_musk s:nytimes)](https://api.tickertick.com/feed?q=(diff%20E:elon_musk%20s:nytimes))| [`(diff E:elon_musk s:nytimes)`](https://api.tickertick.com/search.html?q=(diff%20E:elon_musk%20s:nytimes)) |

### Example API calls

| URL parameters | Semanrics | API call URL | Rendered stories |
|---------------|-----------|-------|---------------------------------|
| `q=tt:amzn`<br>`hours_ago=14400`<br>`n=5` | Get 5 news stories in English about Amazon(`amzn`) from 600 days ago | https://api.tickertick.com/feed?q=tt:amzn&hours_ago=14400&n=5  | [`q=tt:amzn&hours_ago=14400&n=5`](https://api.tickertick.com/search.html?q=tt:amzn&hours_ago=14400&n=5) |
| `q=tt:amzn`<br>`last=1866158884274957563`<br>`n=5` | Get 5 stories about Amazon(`amzn`) older than story with id `1866158884274957563`| https://api.tickertick.com/feed?q=tt:amzn&last=1866158884274957563&n=5  | [`q=tt:amzn&last=1866158884274957563&n=5`](https://api.tickertick.com/search.html?q=tt:amzn&last=1866158884274957563&n=5) |


### The response
The response is a JSON blob consisting of an array of stories in reverse chronological order. Each story has the following fields

| Story field | Description |
|---------------|-----------|
| `id`  | A unique string id of the story. The `id` can be used for pagination as the value of parameter `last`.|
| `title` | The title of the news story. |
| `url` | The url of the news story. |
| `site` | The source website of the news story. |
| `time` | The timestamp of the news story. It's the number of milliseconds since the "Unix epoch", 1970-01-01T00:00:00Z (UTC). The same semantics as `Date.now()` in Javascript. |
| `favicon_url` | The url of the favicon of the source website. |
| `tags` | An array of strings. Each string is the ticker the story is about. This field is presented only when any `tt:` term is in the query.|
| `similar_stories` | An array of strings. Each string is a story id referencing another story in the response. The referenced stories are considered stories similar to this one. This field is optional. |
| `description` | The description of the news story. This field is optional. | 

<!---
> :warning: It's a known issue that some story timestamps are not accurate to varying degrees depending on the source websites. However, the overall accuracy should be fairly good. Most of story timestamps should be at least within one day of real story publication times. The root cause of this issue is a trade-off between backend resource usage and accuracy.
> 
-->

An example response from reqeust URL https://api.tickertick.com/feed?q=(and%20l:en%20tt:aapl)&n=44

```
{
      "id": "6242802208828693749",
      "title": "OnePlus founder Carl Pei quit mid-pandemic to launch an Apple challenger. He envisages an ecosystem of devices with his current startup, Nothing.",
      "url": "https://markets.businessinsider.com/news/stocks/nothing-founder-carl-pei-quit-oneplus-in-pandemic-apple-2021-11",
      "site": "businessinsider.com",
      "time": 1636624740000,
      "favicon_url": "https://static.tickertick.com/website_icons/businessinsider.com.ico",
      "tags": [
        "aapl"
      ],
      "description": "Nothing CEO and founder Carl Pei. Nothing"
    },
    {
      "id": "-7457146849426956238",
      "title": "iCloud for Windows gets a built in password generator",
      "url": "https://www.theverge.com/2021/11/11/22775976/icloud-for-windows-password-generator-prores-proraw",
      "site": "theverge.com",
      "time": 1636623168000,
      "favicon_url": "https://static.tickertick.com/website_icons/theverge.com.ico",
      "tags": [
        "aapl"
      ],
      "similar_stories": [
        "6026104833477490720",
        "-8455375805591907240",
        "4759898561694343437",
        "3814604136583654940",
        "7811723997031023014",
        "-7906149776187926921",
        "4042098602080329618",
        "5670534624353044826"
      ],
      "description": "iCloud for Windows’ password manager in action. | Screenshot: iCloud for Windows A new update for Apple’s iCloud Windows app has added a password generator, as well as ..."
    },
    ...
  "last_id": "-990234403033955973"
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

## Contact

HC Zhu - [mail AT tickertick.com](mailto:mail@tickertick.com) - [@hzhu_](https://twitter.com/hzhu_)

## Acknowledgments

### SpaCy
The backend of TickerTick API uses [spaCy](https://github.com/explosion/spaCy) to process and anaylyze stories.

### MediaWiki API
The backend of TickerTick API uses [MediaWiki API](https://www.wikidata.org/w/api.php) to get entities related to a tickre, e.g. `Elon Musk` for `TSLA`, and `WhatsApp` for `FB`.

### Favicon Grabber
The backend of TickerTick API uses [Favicon Grabber](https://github.com/antongunov/favicongrabber.com) to fetch website favicons.

## API use cases

### TickerTick.com
[TickerTick.com](https://tickertick.com) provides the broadest stock news.
