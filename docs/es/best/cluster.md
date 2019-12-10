# Cluster Organization

As in every system to be designed we have to take some decisions about how the data is going to be stored. The first step that we need to consider is the amount of data we have for 2 reasons:
1. We do not want to use more than 80% of the storage provided in our cluster because that will enforce ElasticSearch to reindex the data between the nodes and make the cluster slower.
2. It is important to know how to organize the indices we are going to create.

## How to organize my indices? 
We suggest organizing them either monthly or daily according to the amount of data you are going to store.
* If you have less than 1GB of data per day it is suggested that you created monthly indices with 1 shard.
* If you have 2-3GB of data per day it is suggested that you creare montly indices with 3-5 shards.
* If you have more data than the 2 cases mentioned above then create daily indices.

## How to decide the amount of shards for my indices?
You should remember that 1 shard should store at most 10GB of data for the current hardware and that shards do not come for free. Consequently your cluster may become slow in indexing or lose connection to ElasticSearch if you tend to use a large number of shards. As a general rule we say that we have a maximum of 750 shards per node in the cluster. Remember that in the number of shards you are setting you should also count that the replica uses the same amount of shards as the master, ie: if you define an index with 1 replica to use 3 shards then the total amound of shards used by this index is 6.

## How can we avoid exceeding the maximum number of shards?
There are cases in which aliases can be used. If you are having daily / monthly indices which have data from many environments (test / dev / prod) you can create aliases to avoid splitting the index to smaller ones. In that case, we can use an alias on the index we want to with the environment.

