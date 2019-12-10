#  Number of shards

Please, be aware that shards come at a price. See https://www.elastic.co/guide/en/elasticsearch/guide/current/kagillion-shards.html for more details. In particular, by default kibana 5.X does not support configuration queries that go over 1000 shards or more. A high number of shards has a negative impact on the performance of the whole cluster.
