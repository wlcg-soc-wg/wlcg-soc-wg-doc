# Elastalert

A useful tool to inspect and alarm specific metrics is Elastalert. It is a service which can be run externally to Elasticsearch. It is used at CERN to get notified about specific conditions which are being monitored, like the size of indices, rate of events or the used space above quota. 

## Installation
Elastalert can be found here:
https://github.com/Yelp/elastalert

CERN can provide an RPM package for it.

## Usage
Documentation can be found here:

http://elastalert.readthedocs.io/en/latest/

## Example usage from our monitoring
Let's say you have an indices which are named ```monitor_usage-<date>```. The document ID (```_id```) is the name of an index, and the documents of type ```disk_usage``` contain (at least) 4 fields, giving the size of that index (```size```), the number of shards ```shards```, the name of the cluster (```cluster```) in which this index lives, and a time stamp (```time```). 

The use case is to check on a weekly basis indices which have a size larger than some threshold. A mail should be send every Monday reporting all indices whoes size exceeds a threshold, taking into account the number of shards. 

A sample configuration for ElastAlert could look like this:

```
# (Required)
# The alert is use when a match is found
alert:
- "email"
from_addr: "elastalert@somewhere.somedomain"
alert_subject: "ElastAlert: index {0} in cluster {1} has size {2}"
alert_subject_args:
- _id
- cluster
- size

#
# check for large indices
description: "large indices found"

# (required, email specific)
# a list of email addresses to send alerts to
email:
- "alert-reciever@somewhere.somedomain"

# Alert when an index is really big

# (Required)
# Rule name, must be unique
name: Large index

# (Required)
# Type of alert.
# the frequency rule type alerts when num_events events occur with timeframe time
type: any

# (Required)
# Index to search, wildcard supported
index: monitor_usage-*

# (Required, frequency specific)
# Alert when this many documents matching the query occur within a timeframe
num_events: 1

timestamp_field: time

# (Required, frequency specific)
# num_events must occur within this amount of time to trigger an alert
timeframe:
  hours: 1

# fields to return
include: ['_id', 'size', 'usage', 'cluster']

realert:
  minutes: 0

aggregation:
  schedule: '0 7 * * 1 *'

# (Required)
# A list of Elasticsearch filters used for find events
# These filters are joined with AND and nested in a filtered query
# For more info: http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/query-dsl.html
filter:
- bool:
   must:
     - range:
         time:
           gte: "now-8h"
     - match:
         _type:
            query: "disk_usage"
            type: "phrase"
     - script:
         script: 
           inline: "doc['size'].value > doc['shards'].value * 100 * 1024 * 1024 * 1024 "
           lang: painless

```
