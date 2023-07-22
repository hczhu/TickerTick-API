## Introduction

Welcome to the TickerTick API.

The TickerTick API provides the latest stock news stories through a powerful query language. The API covers all companies listed in US stock markets ([around 10,000 tickers](https://api.tickertick.com/tickers?n=100000)) and [hundreds of top startups](https://api.tickertick.com/tickers?n=100000&p=.). The source websites of the news stories include around ten thousand websites.

Take a look at an example web app that uses TickerTick API.
- [https://tickertick.com](https://tickertick.com)


### Terms of Use
- The API is free of commercial/non-commercial use.
- All endpoints have a rate limit of __10 requests per minute__ from the same IP address. The service enforces this. More precisely, an IP will be blocked for one minute if more than 10 requests are sent within any 1 minute time window.
- You are welcome to file an issue if you see any problem, like irrelevant stories.

> :warning: An IP address has a rate limit of __10 requests per minute__. DM me on [Twitter](https://twitter.com/hzhu_) if you need a higher request rate.

## Quickstart

Use [this URL](https://api.tickertick.com/feed?q=z:aapl&n=200) to get the 200 latest news stories for ticker `aapl` (Apple Inc.).

`https://api.tickertick.com/feed?q=z:aapl&n=200`

You can replace `aapl` (the ticker) and `200` (the number of news stories to fetch).

### Call TickerTick API in ChatGPT
|  | |
|---------------|-----------|
| ![image](https://github.com/hczhu/TickerTick-API/assets/2289363/d62096ae-1909-40cd-a969-b450b6efa75e) | ![image](https://github.com/hczhu/TickerTick-API/assets/2289363/be9dfa8a-0ca1-4da2-87a3-e45186da6c6c)




## Endpoints
### `GET` https://api.tickertick.com/feed
This endpoint returns a feed of the latest news stories relevant to the query in reverse chronological order.
#### Example request URL
News stories about Apple Inc. (its ticker is __aapl__) - https://api.tickertick.com/feed?q=tt:aapl

[Rendered stock news feed for Apple Inc.](https://tickertick.com/ticker/aapl/feed)

#### Parameters

| Parameter  | Description                     | Options                            |  Example value    |
|----------------|-------------------------------|--------------------------------------------------------|------------|
| q              | The query string       | Any query string in a query language<br>(explained below) | `(or tt:aapl tt:amzn)`  |
| n              | How many news stories to fetch|   Any number between 1 and 1000 |      `42`      |
| last    | A story id for pagination.<br>Fetch news stories older than the story with this id.| A 64-bit integer. Each returned news story has an id. |  `6844326865886118959`       |
| hours_ago | Only return stories from these many number of hours ago or newer.| A 32-bit positive integer. | `2400` (100 days ago) <br> `24000` (1000 days ago)|


#### The query language
The query language is a [context-free language](https://en.wikipedia.org/wiki/Context-free_language) following the grammar below
```
query --> term | (and query_list) | (or query_list) | (diff query query)

query_list --> query query_list | term

term --> tt:any_stock_ticker | TT:any_stock_ticker | s:any_website_domain_name | E:any_entity | T:story_type 

```

#### Operator semantics
| Operator | Semantics|
|---------------|-----------|
| `(and query_list)` | Request news stories matching all queries in `query_list`|
| `(or query_list)` | Request news stories matching any query in `query_list`|
| `(diff query1 query2)` | Request news stories matching `query1` but not `query2`|


#### Term semantics
| Term | Semantics | Examples |
|---------------|-----------|---|
| `z:stock_ticker`| Request news stories about `stock_ticker`. | `z:aapl`  <br>`z:tsla` |
| `tt:stock_ticker`| Request more broad news stories about `stock_ticker` (more news stories than `z:stock_ticker`)  | `tt:aapl`  <br>`tt:tsla`  |
| `s:domain_name`| Request news stories from websites on domain `domain_name` <br> (`domain_name` shouldn't contain '.' or '/')  | `s:wsj` <br> `s:cnbc`|
| `E:any_entity`| Request news stories with titles semantically matching `any_entity`. <br> (replace any whitespace in `any_entity` by `_` )<br> (`any_entity` should be in lower case)  | `E:shiba_inu`  <br> `E:rent_the_runway` <br> `E:elon_musk`  <br> `E:zoom` |
| `T:story_type`| Request news stories of a specific type. See [the list of all story types](https://github.com/hczhu/TickerTick-API/blob/master/README.md#story-types).  | `T:curated`  <br> `T:sec` |

#### Story Types
| Term | Story type| Example query |
|---------------|-----------|----|
| T:curated | News stories from a curated list of [top financial/technology news sources](https://github.com/hczhu/TickerTick-API/blob/master/docs/top-news-sources.txt) | [T:curated](https://api.tickertick.com/feed?q=T:curated)
| T:earning | Company earnings news (e.g. presentations, transcripts) | [T:earning](https://api.tickertick.com/feed?q=T:earning) 
| T:market | Stock market news | [T:market](https://api.tickertick.com/feed?q=T:market)
| T:sec | SEC filings | [T:sec](https://api.tickertick.com/feed?q=T:sec)
| T:sec_fin | Quarterly/annual financial reports | [T:sec_fin](https://api.tickertick.com/feed?q=T:sec_fin)
| T:trade | Trading news | [T:trade](https://api.tickertick.com/feed?q=T:trade)
| T:ugc | News stories from a curated list of [user-generated content platforms](https://github.com/hczhu/TickerTick-API/blob/master/docs/ugc-sources.txt), e.g. Reddit.  | [T:ugc](https://api.tickertick.com/feed?q=T:ugc)
| T:analysis | Stock analysis articles from [a curated list of sources](https://github.com/hczhu/TickerTick-API/blob/master/docs/top-news-sources.txt) | [T:analysis](https://api.tickertick.com/feed?q=T:analysis)
| T:industry | Industry publications from [a curated list of sources](https://github.com/hczhu/TickerTick-API/blob/master/docs/industry-publications.txt) | [T:industry](https://api.tickertick.com/feed?q=T:industry)

#### Example queries
| Example query | Semantics | API call URL | Rendered stories |
|---------------|-----------|-------|---------------------------------|
| `(and tt:aapl s:sec)`  | SEC filings from Apple Inc. (ticker: aapl)| [https://api.tickertick.com/feed?q=(and tt:aapl s:sec)](https://api.tickertick.com/feed?q=(and%20tt:aapl%20s:sec))      | [`(and tt:aapl s:sec)`](https://api.tickertick.com/search.html?q=(and%20tt:aapl%20s:sec)) |
| `(or tt:meta tt:aapl tt:amzn tt:nflx tt:goog)` | News stories about [FAANG stocks]([https://www.investopedia.com/terms/f/fang-stocks-fb-amzn.asp](https://www.investopedia.com/terms/f/faang-stocks.asp)) | [https://api.tickertick.com/feed?q=(or tt:meta tt:aapl tt:amzn tt:nflx tt:goog)](https://api.tickertick.com/feed?q=(or%20tt:meta%20tt:aapl%20tt:amzn%20tt:nflx%20tt:goog)) | [`(or tt:meta tt:aapl tt:amzn tt:nflx tt:goog)`](https://api.tickertick.com/search.html?q=(or%20tt:meta%20tt:aapl%20tt:amzn%20tt:nflx%20tt:goog)) |
| `(and (or tt:meta tt:goog) s:reddit)` | News stories about Meta (`meta`) and Google (`goog`) from `reddit.com` | [https://api.tickertick.com/feed?q=(and (or tt:meta tt:goog) s:reddit)](https://api.tickertick.com/feed?q=(and%20(or%20tt:meta%20tt:goog)%20s:reddit))|[`(and (or tt:meta tt:goog) s:reddit)`](https://api.tickertick.com/search.html?q=(and%20(or%20tt:meta%20tt:goog)%20s:reddit)) |
| `(diff (or tt:meta tt:goog) s:reddit)` | News stories about Meta (`meta`) and Google (`goog`) not from Reddit.com | [https://api.tickertick.com/feed?q=(diff (or tt:meta tt:goog) s:reddit)](https://api.tickertick.com/feed?q=(diff%20(or%20tt:meta%20tt:goog)%20s:reddit))|[`(diff (or tt:meta tt:goog) s:reddit)`](https://api.tickertick.com/search.html?q=(diff%20(or%20tt:meta%20tt:goog)%20s:reddit)) |
| `(diff E:elon_musk s:nytimes)` | Stories with `Elon Musk` in titles not from The New York Times | [https://api.tickertick.com/feed?q=(diff E:elon_musk s:nytimes)](https://api.tickertick.com/feed?q=(diff%20E:elon_musk%20s:nytimes))| [`(diff E:elon_musk s:nytimes)`](https://api.tickertick.com/search.html?q=(diff%20E:elon_musk%20s:nytimes)) |

#### Example API calls
| URL parameters | Semanrics | API call URL | Rendered stories |
|---------------|-----------|-------|---------------------------------|
| `q=tt:amzn`<br>`hours_ago=14400`<br>`n=5` | Get 5 news stories about Amazon(`amzn`) from 600 days ago | https://api.tickertick.com/feed?q=tt:amzn&hours_ago=14400&n=5  | [`q=tt:amzn&hours_ago=14400&n=5`](https://api.tickertick.com/search.html?q=tt:amzn&hours_ago=14400&n=5) |
| `q=tt:amzn`<br>`last=1866158884274957563`<br>`n=5` | Get 5 stories about Amazon(`amzn`) older than story with id `1866158884274957563`| https://api.tickertick.com/feed?q=tt:amzn&last=1866158884274957563&n=5  | [`q=tt:amzn&last=1866158884274957563&n=5`](https://api.tickertick.com/search.html?q=tt:amzn&last=1866158884274957563&n=5) |


#### Response
The response is a JSON blob consisting of an array of stories in reverse chronological order. Each story has the following fields

| Story field | Description |
|---------------|-----------|
| `id`  | A unique string id of the story. The `id` can be used for pagination as the value of the parameter `last`.|
| `title` | The title of the news story. |
| `url` | The url of the news story. |
| `site` | The source website of the news story. |
| `time` | The timestamp of the news story. It's the number of milliseconds since the "Unix epoch", 1970-01-01T00:00:00Z (UTC). The same semantics as `Date.now()` in Javascript. |
| `favicon_url` | The url of the favicon of the source website. |
| `tags` | An array of strings. Each string is the ticker for which the story is. This field is presented only when any `tt:` term is in the query.|
| `similar_stories` | An array of strings. Each string is a story ID referencing another story in the response. The referenced stories are considered stories similar to this one. This field is optional. |
| `description` | The description of the news story. This field is optional. | 

<!---
> :warning: It's a known issue that some story timestamps are not accurate to varying degrees depending on the source websites. However, the overall accuracy should be fairly good. Most of story timestamps should be at least within one day of real story publication times. The root cause of this issue is a trade-off between backend resource usage and accuracy.
> 
-->

An example response from request URL [https://api.tickertick.com/feed?q=(and T:curated tt:aapl)&n=30](https://api.tickertick.com/feed?q=(and%20T:curated%20tt:aapl)&n=30)

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


#### Parameters
| Parameter  | Description                     | Options                            |  Example value    |
|----------------|-------------------------------|--------------------------------------------------------|------------|
| p              | The query string to match the company name or the stock ticker.      | Any string.| `Tesl`  |
| n            | How many tickers to return at most.     | Any integer.| `4`  |

#### Response
The returned result is a JSON string consisting of all matched stock tickers.

#### Example
Search for any tickers matching `Ama` - https://api.tickertick.com/tickers?p=Ama&n=2
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
Search for any tickers matching `aa` - https://api.tickertick.com/tickers?p=aa&n=2
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

## Contact

HC Zhu - [mail AT tickertick.com](mailto:mail@tickertick.com) - [@hzhu_](https://twitter.com/hzhu_)

## Acknowledgments

### SpaCy
The backend of TickerTick API uses [spaCy](https://github.com/explosion/spaCy) to process and analyze stories.

### MediaWiki API
The backend of TickerTick API uses [MediaWiki API](https://www.wikidata.org/w/api.php) to get entities related to a ticker, e.g. `Elon Musk` for `TSLA`, and `WhatsApp` for `FB`.

### Favicon Grabber
The backend of TickerTick API uses [Favicon Grabber](https://github.com/antongunov/favicongrabber.com) to fetch website favicons.

## API use cases

### TickerTick.com
[TickerTick.com](https://tickertick.com) provides the broadest stock news.

## Big Tech News Twitter bot
[Big Tech News](https://twitter.com/BigTech_News) uses TickerTick API to fetch news about big tech.

## Trading Saga
[Trading Saga](https://tradingsaga.com/) is a quick and immersive trading game and uses TickerTick API to fetch stock news.
