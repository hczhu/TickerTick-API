## Retrieve Most Important News Stories

The running example below retrieves the most important news for `$META` and `$AMZN`
1. Use [the following query](https://api.tickertick.com/feed?n=200&q=(diff%20(or%20(or%20(or%20TT:amzn%20TT:meta)%20(and%20tag:_best_%20(or%20tt:meta%20tt:amzn)%20(or%20T:fin_news%20T:analysis%20T:industry))))%20(or%20T:trade%20T:market%20T:ugc))) to retrieve candidate news stories. Note that there is a special term `tag:_best_`.
```
(diff (or (or (or TT:meta TT:amzn) 
       (and tag:_best_ (or tt:meta tt:amzn) (or T:fin_news T:analysis T:industry)))) 
 (or T:trade T:market T:ugc))
```
2. Look for the stories in the response which have a `similar_stories` section. A story and all its similar stories form a cluster of stories. The story is called the primary story of the cluster.
3. If a cluster has more than 3 (this is tunable) stories, the primary story is considered the most important.
4. If a cluster contains any story with a `best` tag, the story with a `best` tag is considered most important. 
For example, the following story satisfies the two criteria above 
```
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
      ],
    },
```
