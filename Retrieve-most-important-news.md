## A simple way to retrieve most important news

The running example below retrieves the most important news for $META and $AMZN
1. Use [the following query](https://api.tickertick.com/feed?n=200&q=(diff%20(or%20(or%20(or%20TT:amzn%20TT:meta)%20(and%20tag:best%20(or%20tt:meta%20tt:amzn)%20(or%20T:fin_news%20T:analysis%20T:industry))))%20(or%20T:trade%20T:market%20T:ugc))) to retrieve candidate news stories. Note that this is a special term `tag:best`.
```
(diff (or (or (or TT:meta TT:amzn) 
       (and tag:best (or tt:meta tt:amzn) (or T:fin_news T:analysis T:industry)))) 
 (or T:trade T:market T:ugc))
```
2. Look for the stories in the response which have a `similar_stories` section. A story and all its similar stories form a cluster of stories. The story is called the primary story of the cluster.
3. If a cluster has more than 5 stories, the primary story is considered the most important.
4. If a cluster contains any story with a `best` tag, the story with a `best` tag is considered most important.
