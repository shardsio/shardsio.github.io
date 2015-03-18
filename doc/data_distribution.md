---
layout: index
title: Data Distribution
---
### Data Distribution

When distributing a table between several shards there are three basic options you have:

- replicate data on all shards so every shard contains an identical copy of table
- distribute data between shards by some key so a single shard contains records having the same value of a distribution key
- distribute data evenly

It's clear that big quickly growing tables have to be distributed and small static tables have to be replicated.

![small static table](/images/shards5.png)

The goal of keeping small static tables on each shard is to offload as much data processing operations on shards as possible.
So if you need to join some big resultset with small table it's a good idea to do it on shards in order to archieve more parallelization.

The style of distribution also depends if you wish to keep a related data on a single node or you want data distributed as evenly as possible.

For example when we store data about customers most probably it's better to have all data about certain customer on the same shard.
Thus all operations with account can be made within a single database with no cross-sharding overhead.
In such cases we need to distribute data by some field like *customer_id*, *user_name* or similar.

![distribution by key](/images/shards6.png)

Usually it's better to apply some hash function over such field in order to avoid an effect of excess of some values.
Basically if you distribute data by *user_name* most probably there will be much more usernames starting with A letter than with Y.
Hashing the username will help to make a distribution between shards more even. 
Though you still may face the issue of super active users which will make one shard much bigger than another ones.

![round robin distribution](/images/shards7.png)

If you want as uniform shards as possible you need to pick the third option 
and put data more or less evenly on each node.
This option is suitable for batch operations over big data sets 
since it provides more parallelization over single chunk of data independently on the key.


