# Keyword vs text

In ES 5.X there are two datatypes that deal with strings: `keyword` and `text`. When a new string field arrives, by default elasticsearch will store it both as a string and as a text. This can be a waste of space and memory if only one of the two types are needed. The first one will take the whole field as a single token, whereas the text will split it on every single word. Keywords can be used for aggregations, whereas text cannot. See <https://www.elastic.co/guide/en/elasticsearch/reference/5.4/keyword.html> and <https://www.elastic.co/guide/en/elasticsearch/reference/5.4/text.html>


If you want all the strings to be mapped as keywords by default, the easiest solution is to upload a template with a dynamic mapping like

```
{
    "template" : "*",
    "mappings" : {
         "_default_" :{ 
            "dynamic_templates": [  {
                "strings": {
                    "match_mapping_type": "string",
                    "mapping": {
                        "type": "keyword",
                        "ignore_above": 256
                    }
                }
              }
            ] 
         }
  }
}
```

