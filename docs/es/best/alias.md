# Aliases
Aliases can be used for grouping specific data that we want to view. To configure an alias you need to add this section to your template file under templates folder.
From there you can define the name of the index and the date so the pattern is index_name-date or in whichever way you have named your indices.

```
    "aliases": { "index_name-today":{},
                 "index_name-today_prod":{"filter":{"term":{"environment":"PROD"}}},
                 "index_name-today_dev":{"filter":{"term":{"environment":"DEV"}}},
                 "index_name-last7":{},
                 "index_name-last7_prod":{"filter":{"term":{"_environment":"PROD"}}},
                 "index_name-last7_dev":{"filter":{"term":{"environment":"DEV"}}}
    }
```

As we see, we are creating 6 aliases here, one which has today data (to be noted that if your logs do not have UTC timestamp then the today index will start at 1am during winter and at 2am during summer) and then from that index we just do a small query with the term environment (you should change your variable name and the value if they are different) and with the environment we want to retrieve. Same happens for the last7 indices. Now how we are sure that we are getting the data we request. In our curator we have to configure the rule as below:

```
  2:
    action: alias
    description: Remove indices from today alias
    options:
      name: index_name-today 
      extra_settings:
      timeout_override:
      continue_if_exception: False
      disable_action: False
      ignore_empty_list: True

    remove:
      filters:
        - filtertype: age
          direction: older
          unit: days
          unit_count: 1
          source: name
          timestring:  '%Y.%m.%d'
        - filtertype: pattern
          kind: prefix
          value: index_name
          exclude:
        - filtertype: alias
          aliases: index_name-today 
          exclude: False

``` 

* Note: __Change the timestring in case you have indices with dashes__

And one example with the environment result:

```
  7:
    action: alias
    description: Remove indices from today alias prod environment
    options:
      name: index_name-today_prod
      extra_settings:
      timeout_override:
      continue_if_exception: False
      disable_action: False
      ignore_empty_list: True

    remove:
      filters:
        - filtertype: age
          direction: older
          unit: days
          unit_count: 1
          source: name
          timestring:  '%Y.%m.%d'
        - filtertype: pattern
          kind: prefix
          value: index_name
          exclude:
        - filtertype: alias
          aliases: index_name-today_prod
          exclude: False
```

For the last7 indices you need to create the same aliases but the unit_count should be 7 instead of 1.
