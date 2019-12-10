# Disabling norms

By default, elasticsearch will calculate a score for the relevance of documents for queries. This is not needed for normal filters and aggregations. If your use case does not need norms, it is often better to disable them. See https://www.elastic.co/guide/en/elasticsearch/reference/5.4/norms.html

To disable norms, upload the following template
```
 {
    "template" : "*",
    "mappings": {
      "_default_": {
        "_all": { "enabled" : true, "omit_norms" : true },
        "dynamic_templates": [ {
          "omit_norms": {
            "match_mapping_type": "*",
            "mapping": {
              "norms": "false"

            }
          }
        } ]
      }
    }
}

```
