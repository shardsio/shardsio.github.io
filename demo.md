---
layout: demo
title: Demo
---

After requesting a demo access you will receive an email with credential to access a demo **shards.io** cluster.
The cluster will be available for 3 days.

### Cluster description

Each demo cluster is installed on a single server with 2 Core processor, 4 GB RAM and 60 GB of SSD.
Cluster consists of 5 databases: *demo*, *demo\_00*, *demo\_01*, *demo\_02*, *demo\_03*.

The *demo* database is an entry point you usually want to connect to with standard Postgresql tools like pgAdmin or psql.
This is a usual Postgresql database and you are free to use it for storing your data and running queries over it.

Databases *demo_NN* are shards. These are also normal Postgresql databases but intended for distributed data.

### Samples

There are two preinstalled sample applications represinting the possible ways of organising data.
Data are provided by Seznam.cz and CCS companies for [Enterprise Data Hackathon](http://enterprise.hackathon.bi)

### Seznam.cz

Data from Seznam.cz is a good example of semi-structured data represented in JSON format.
Since it's a document like structures we store each JSON in a table **seznam.docs**. 
The table is distributed over shards evenly. So each shard contains only part of the entire dataset.

Here are the sample queries you can run on *demo* database in order to get information about seznam documents from all shards.

**Number of documents on each shard**

``` sql
WITH v(v) AS (
SELECT shards.from_all($$
    SELECT current_database(), COUNT(*) FROM seznam.docs
$$)
)
SELECT 
v->>'current_database' shard_name,
(v->>'count')::INTEGER records_number
FROM v
```

     shard_name | records_number 
    ------------+----------------
     demo_00    |          25133
     demo_01    |          25157
     demo_02    |          24859
     demo_03    |          24851
    (4 rows)

**10 recent queries**

``` sql
WITH v(v) AS (
SELECT shards.from_all($$
    SELECT doc FROM seznam.docs
    ORDER BY to_timestamp((doc->>'time')::FLOAT) DESC
    LIMIT 10
$$)
)
SELECT v->'doc' FROM v
ORDER BY to_timestamp((v->'doc'->>'time')::FLOAT) DESC
LIMIT 10
```

          query       |            time            
    ------------------+----------------------------
     5e0f665c769d5765 | 1971-04-17 21:30:08.269+03
     e70d81f14061a701 | 1971-04-17 21:30:03.499+03
     71e0a53823d5a78c | 1971-04-17 21:29:57.059+03
     df407139b337cb56 | 1971-04-17 21:29:55.21+03
     36313a9278313821 | 1971-04-17 21:29:51.15+03
     1abc2e0d71b8cca0 | 1971-04-17 21:29:50.429+03
     44a373b6bc1f7ac5 | 1971-04-17 21:29:32.659+03
     3b83c8783f2ccaf3 | 1971-04-17 21:29:31.17+03
     c91c5891c88e49f5 | 1971-04-17 21:29:25.39+03
     2693ea9a3735d683 | 1971-04-17 21:29:24.329+03
    (10 rows)

**Transform JSON document into a classic table view**

``` sql
WITH v(v) AS (
SELECT shards.from_all($$
    SELECT * FROM seznam.docs
    WHERE doc->>'long_session' = '6af7fbbde96df8b2'
$$)
)
SELECT
v->'doc'->>'long_session' long_session,
v->'doc'->>'autocompleted' autocompleted,
v->'doc'->>'ip' ip,
v->'doc'->>'page' page,
v->'doc'->>'query' query,
v->'doc'->>'wordCount' wordCount,
v->'doc'->>'short_session' short_session,
v->'doc'->>'time' ttime,
v->'doc'->>'other_clicks' other_clicks,
json_array_elements(v->'doc'->'result')->>'url' url,
json_array_elements(v->'doc'->'result')->>'domain' ddomain,
json_array_elements(v->'doc'->'result')->>'l2domain' l2domain,
json_array_elements(v->'doc'->'result')->'clicks' clicks
FROM v
```

       long_session   | autocompleted |        ip        | page |      query       | wordcount |  short_session   |    ttime    | other_clicks |       url        |     ddomain      |     l2domain     |     clicks     
    ------------------+---------------+------------------+------+------------------+-----------+------------------+-------------+--------------+------------------+------------------+------------------+----------------
     6af7fbbde96df8b2 | false         | 4ae4cacec9622238 | 0    | 2f1c334c9b54bef3 | 2         | 06e013d6a5580de0 | 40739373.59 |              | 7cb34456d21aafeb | 0a41552f1b6a8ddf | 2443e1f059973b93 | [40739377.302]
     6af7fbbde96df8b2 | false         | 4ae4cacec9622238 | 0    | 2f1c334c9b54bef3 | 2         | 06e013d6a5580de0 | 40739373.59 |              | f56edfc126d14fd0 | d14b23c951187106 | 9392259ae7e186b7 | 
     6af7fbbde96df8b2 | false         | 4ae4cacec9622238 | 0    | 2f1c334c9b54bef3 | 2         | 06e013d6a5580de0 | 40739373.59 |              | 45886fbd33dc6d21 | c4281d6f23e1bbbf | ace47c62285475ee | [40739391.855]
     6af7fbbde96df8b2 | false         | 4ae4cacec9622238 | 0    | 2f1c334c9b54bef3 | 2         | 06e013d6a5580de0 | 40739373.59 |              | 175a5f99b58df09d | d89031af5948eced | c81b911ebdb7605c | 
     6af7fbbde96df8b2 | false         | 4ae4cacec9622238 | 0    | 2f1c334c9b54bef3 | 2         | 06e013d6a5580de0 | 40739373.59 |              | 5c7b17fd5b9fff31 | 40b842c46ca235c3 | 5780d9f6a5d9807c | 
     6af7fbbde96df8b2 | false         | 4ae4cacec9622238 | 0    | 2f1c334c9b54bef3 | 2         | 06e013d6a5580de0 | 40739373.59 |              | 9eb188476a339e37 | 924edd037056cc0a | 37ba2bb3ffa35a52 | [40739426.948]
     6af7fbbde96df8b2 | false         | 4ae4cacec9622238 | 0    | 2f1c334c9b54bef3 | 2         | 06e013d6a5580de0 | 40739373.59 |              | 41fb81192aedf24a | 3493ffadd89d3159 | e9222a76d513dda3 | [40739491.678]
     6af7fbbde96df8b2 | false         | 4ae4cacec9622238 | 0    | 2f1c334c9b54bef3 | 2         | 06e013d6a5580de0 | 40739373.59 |              | 8f8e13827f797240 | dfc5d45de3ee46ac | 7f77d8676335396e | [40739480.876]
     6af7fbbde96df8b2 | false         | 4ae4cacec9622238 | 0    | 2f1c334c9b54bef3 | 2         | 06e013d6a5580de0 | 40739373.59 |              | a1f230d9e761e88b | d6ea9cef60d7235a | fa9bcce28ca9638b | 
     6af7fbbde96df8b2 | false         | 4ae4cacec9622238 | 0    | 2f1c334c9b54bef3 | 2         | 06e013d6a5580de0 | 40739373.59 |              | fc4715d89261c072 | ec50d490179b3800 | 1e869df45b0989e9 | 
    (10 rows)


### CCS

A sample data from CCS is a good example how could classic relational data fit into distributed model.
There are 4 tables:

- ccs.customers
- ccs.products
- ccs.gasstations
- ccs.transactions

The first three tables are "replicated", i.e. there is an identical copy of a table on each shard.
And the table *ccs.transactions* is distributed by *customer_id* field. So transactions from each customer are stored on the same shard.

Here is an example of standard question answered with SQL over distributed database

**Find the customer with the biggest number of transactions**

``` sql
WITH v(v) AS (
SELECT shards.from_all($$
    SELECT 
        t.customer_id, 
        COUNT(*) AS transactions,
        c.currency, 
        c.segment
    FROM ccs.transactions t
    JOIN ccs.customers c ON c.customer_id = t.customer_id
    GROUP BY t.customer_id, c.currency, c.segment
    ORDER BY transactions DESC
    LIMIT 1
$$)
)
SELECT
v->>'customer_id' AS cusomer_id,
(v->>'transactions')::INTEGER AS transactions,
v->>'currency' AS currency,
v->>'segment' AS segment
FROM v
ORDER BY transactions DESC
LIMIT 1
```

     cusomer_id | transactions | currency  | segment 
    ------------+--------------+-----------+---------
     15064      |            3 | CZK       | LAM
    (1 row)
