# Number of indices

The number and name of the indices is one of the first design decisions that have to be taken. If the data has some timestamps, it is good practice to create time based indices (hourly, daily, monthly, yearly ...), for example `entrypoint-mylogs-2017.10.31`. The granularity of the index should be decided based on the amount of data expected, and the type of queries. For instance, if most queries will access a month of data, it is often better to create monthly indices.
