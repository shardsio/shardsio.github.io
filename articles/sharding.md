---
layout: doc
title: Sharding Overview
---
### Sharding Overview

Sharding is a common technique used for horizontal database scaling. 
The main idea behind sharding is splitting a database to several ones and keeping them on different hosts.
More hosts can handle bigger load, also they can process data faster since the parallelization of work.
There are several disadvantates of this approach such as more complex administration and some limitations of distributed data model.
We will address this questinos later in more details, 
but in general they are solvable and shouldn't stop you from your way to the shared nothing architecture of your database.

Let's consider different types of application design when using sharding.

The first one is the most obvious one: *Client side sharding or sharding aware application*.
This means that you have several databases and your application knows about that, 
moreover application distributes the data between shards itself.

Such design usually requires writing some kind of frameworks on application side for working with shards.
When more services working with your database appear a changing shards configuration can become a problem, 
since you need to change configuration for all of your services.

The second type is *Transparent sharding or MPP database*.
Such solutions suppose that application communicates with database as it wasn't distributed 
and a database engine handles the distribution logic itself.
Sounds great but there are several gotchas as well. 
In order to supply ACID in MPP database there should exist a global transaction manager responsible for locking.
And such manager becomes a bottleneck in case of big transactional load.
Another disadvantage of known open source implementations based on Postgresql is delay with releases in comparison with the parent.
It means that you have to work with several years old postgresql core, not always having all neccessary patches and missing new cool features.
We will not consider commercial MPP database vendors here but in general with them the following rule works pretty well: big data require big money.

And the third option is *Map/Reduce approach*.
It means that application still doesn't know about distributed nature of database infrastructure.
But there exists a layer of logic in database which is responsible for splitting queries between shards 
and post processing the obtained result before returning it to an application.
This is the most interesting option since it doesn't have scalability issues like MPP 
and it ensures better design of application especially if you like service oriented architectures. 
The data processing logic stays inside the database where you have all the neccessary tools for that like SQL.

Surprisingly doing map/reduce in SQL is extremely easy. 
For example let's take a simple table with some stats about web sites with the following structure:
    
    ``` yaml
    table: web_stat
    columns:
        - url: text
        - visited_at: timestamptz
    ```

In single instance database in order to get 10 most popular URLs we would run query like that:

    ``` sql
    SELECT 
        url, 
        COUNT(*) cnt 
    FROM web_stat
    GROUP BY url
    ORDER BY cnt DESC
    LIMIT 10;
    ```

Now with distributed database it looks a bit longer:

    ``` sql
    WITH s(v) AS (
    SELECT shards.from_all($$
        SELECT 
            url, 
            COUNT(*) cnt 
        FROM web_stat
        GROUP BY url
    $$))
    SELECT 
        v->>'url' url, 
        SUM((v->>'cnt')::INTEGER) cnt 
    FROM s
    GROUP BY url
    ORDER BY cnt DESC
    LIMIT 10
    ```

Here we see that first we run usual SQL for counting urls on each shard.
Results returned back from shards as a single dataset serialized in json format.
So we get data out of json and summarize counts from each shard and do the final sorting.


