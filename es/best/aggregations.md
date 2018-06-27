# Aggregating statistics

If there are too many documents on a particular set of indices, it might be a good idea to collect statistics of the most common fields and store those statistics on a separate index. That would speed up the queries and visualizations.

This document describes how the approach of gathering statistics was used to improve the performance of the http access monitoring of the elasticsearch service. For the sake of simplicity, it will use logstash for scheduling, manipulating and storing the data.

## 1. Identify fields to aggregate

The raw documents describe the accesses to a port on a host by a user. The document looks like:
```
* timestamp 	October 14th 2016, 14:45:03.000
* type 	     logstash_apacheaccess
* auth    	  es-atlas
* bytes 	    134
* clientip 	 188.185.164.175
* cluster   	public
* count 	    1
* host 	     ess003.cern.ch
* httpversion   1.1
* input_type    log
* port 	     9,203
* request 	  /atlas_rucio-popularity-*/_search
* response      200
* verb 	     POST
```

The existing plots were collecting data based on timestamp, auth, cluster, port and response, so let's start with those ones, and assume that the other fields are not important for the statistics.

## 2. Create query to aggregate

The easiest way to aggregate the data is to use the aggregation from elasticsearch itself. First, we have to select the timeperiod, and then aggregate them.  A query like the following will do the trick for us
```
[perfmon@localhost ~]$ cat search_access.json
 {
  "query": {
    "filtered": {
      "filter": { "range": { "@timestamp": {"gte": "now-2h/h","lte": "now", "format": "epoch_millis" } }  }
    }
  },
  "size": 0,
  "aggs": { "2": {
      "date_histogram": { "field": "@timestamp", "interval": "1h",  "min_doc_count": 1, "extended_bounds": {
          "min": "now-1h/h","max": "now" } },
      "aggs": { "3":{
        "terms": {"field": "cluster", "size": 100 , "order": {"_count": "desc" }},
        "aggs" : { "4":{
          "terms": {"field": "auth", "size":100, "order":{"_count":"desc"}},
          "aggs": { "5":  {
            "terms": {"field":"port", "size":100, "order":{"_count":"desc"}} ,
            "aggs":  {"6": {
              "terms": {"field":"response", "size":100, "order":{"_count":"desc"}} }
                     }
             }
        }
      }
    }
  }
}}}
}

```
Note a couple of important things on this query:
* The time is relative to the current time, and rounded to the hour ('now-2h/h')
* The results of the search are not returned back. Only the aggregations are sent back ("size":0)
* For each aggregation, there is the maximum number of entries, in this example 100 ( "size":100)

This query can be executed and it will return something like:
```
[perfmon@localhost ~]$ curl "http://es-perfmon.cern.ch/es/perfmon_logstash*/logstash_apacheaccess/_search?pretty" -d "@/etc/logstash/conf.d_access/search_access.json"
{
  "took" : 769,
  "timed_out" : false,
  "_shards" : {
    "total" : 63,
    "successful" : 63,
    "failed" : 0
  },
  "hits" : {
    "total" : 1840056,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "2" : {
      "buckets" : [ {
        "key_as_string" : "2016-10-14T13:00:00.000+02:00",
        "key" : 1476442800000,
        "doc_count" : 781868,
        "3" : {
          "doc_count_error_upper_bound" : 0,
          "sum_other_doc_count" : 0,
          "buckets" : [ {
            "key" : "itdb",
            "doc_count" : 357270,
            ...

```
Note that for every aggregation, there are two fields, doc_count and sum_other_doc_count, that returns the total number of documents on that aggregation, and the ones that are not within the limits specified.
## 3. Run the query periodically
Now that the query has been selected, it has to be scheduled to run with a certain frequency. The following logstash configuration will do the trick, querying every ten minutes

```
input {
   exec {
      command => 'curl "http://localhost:9200/perfmon_logstash*/logstash_apacheaccess/_search" -d "@/etc/logstash/conf.d_access/search_access.json" 2> /dev/null'
      interval => 600
  }
}
```

## 4. Transform the result of the query

The result of the previous exercise is a single document. The next step is to break it up into smaller pieces, with one document per subaggregation. That can be done quite easily with the 'json', 'split' and 'mutate' filters of logstash:

```
filter {
   json{ source => "message"
        remove_field => ["message", "host", "command", "hits", "_shards", "took", "timeout", "timed_out", "@version"]
        add_field => {"stats_host" => "psaiz01.cern.ch" }
      }
   split { field => "aggregations[2][buckets]"
           target => "subbuckets"
           remove_field => ["aggregations"]
 }
   split {  field => "subbuckets[3][buckets]"
            add_field => { "time" => "%{subbuckets[key]}" }
            target =>  "sub_cluster"
            remove_field => ["subbuckets"]
 }
   split {  field => "sub_cluster[4][buckets]"
            add_field => { "cluster" => "%{sub_cluster[key]}" }
            target =>  "sub_user"
            remove_field => ["sub_cluster"]
 }
   split {  field => "sub_user[5][buckets]"
            add_field => { "auth" => "%{sub_user[key]}" }
            target =>  "sub_port"
            remove_field => ["sub_user"]
 }
   split {  field => "sub_port[6][buckets]"
            add_field => { "port" => "%{sub_port[key]}" }
            target =>  "sub_response"
            remove_field => ["sub_port"]
 }
  mutate { rename => {  "sub_response[key]" => "response"
                        "sub_response[doc_count]" => "count" }
           remove_field =>  ["sub_response"]
  }
}

```
Things to note here:
* The first filter, json, transform the message into json format, drops some unnecessary fields, and adds one with the current host
* Each of the split filters creates individual documents for the corresponding subaggregation
* Finally, the mutate filter is used to ensure that the fields of the stats documents have the same name than the ones in the raw documents. That will make the creation of dashboards much easier

## 4. Insert results in another index
Since logstash has been used for the massaging of the data, let's use it also for inserting the results:
```
output {
  elasticsearch {
      hosts => "https://es-entrypoint.cern.ch:9203"
      user => "some_user"
      password => "some_secret"
      manage_template => false
      index => "perfmon_apacheaccess_stats-%{+YYYY}"
      document_type => "logstash_apacheaccess_stats"
      document_id => "%{time}_%{cluster}_%{auth}_%{port}_%{response}"
    }
}
```
Note that the document_id has been specified, and it includes all the fields selected for the statistics. This way, we ensure that, if the same period is calculated multiple times, the results will overwrite each other.

Please also note that the use of port 9203 is deprecated. Use https://es-entrypoint.cern.ch/es instead. It is still provided for backward compatibility. Also issues have been seen with older versions of logstash when the `/es` path was used.

## 5. Change kibana visualizations to use statistics

Now that there is a new index with the statistics, the last step is to change the visualizations in kibana to search in the new index, and to plot the sum of the field 'count' instead of just counting the number of documents.

## 6. Closing/deleting old indices

The statistics can be used to generate several different plots. Sometimes it is still necessary to go back to the raw data. In this case in particular, if we want to see the actual requests, they are kept only on the raw data. Having this in mind, and for this particular case, the approach taken was to use curator to close the raw indices after one week, and to delete them after two weeks. See the dedicated chapter on curator.

## 7. Running in high availability

Logstash is usually configured to run on a single machine, which is a single point of failure. A possible solution to this is to configure the service on several machines. Then, each node checks if the statistics have been generated recently, and who did it. If the statistics are not being generated the node will start the logstash service. If the statistics are being generated by another node, the node will stop logstash.

## 8. Running logstash from acron
Logstash is made to be run as a continuously running service. Therefore, it is not straight-forward to run it within acron which implies a one-shot operation. Also, using acron is not a way to grant high availability
because the acron server itself is a single point of failure. If not dedicated machine is available to run logstash it may be useful though to go down this road. You need to create a wrapper script which you execute
from acron periodically.
* use curl directly to retrieve the input data, and store it in a temporary file
* Use the stdin input plugin for ES like this:
```bash
input {
   stdin {
      type => "stdin-type"
   }
}
```
* run logstash like this:
```bash
logstash  -f /path/to/logstash/configfile/logstash_access.conf < $tmpfile
```
Note that logstash is not available on aiadm nor on lxplus. You have to install the proper version locally in your home directory. Also make sure that logstash_access.conf is protected so that any access credentials in there
are not readable by anybody but you.
