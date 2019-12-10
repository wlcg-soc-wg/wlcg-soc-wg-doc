# Slow logs
It is possible to configure elasticsearch to log all the queries and inserts that take more than a certain threshold. This is very useful while debugging applications. This feature can be enabled on an index-by-index level, so the easiest way is to include it on the template. The following snippet would do the trick:

```
 {
    "order" : 1,
    "template" : "perfmon_*",
    "settings" : { "index.number_of_shards" : "1"  ,
                   "index.search.slowlog.threshold.fetch.warn" : "1s",
                   "index.search.slowlog.threshold.query.warn" : "10s",
                   "indexing.slowlog.level":"info",
                   "indexing.slowlog.threshold.index.warn" : "10s",
                   "indexing.slowlog.threshold.index.info" : "5s" }
}
```
