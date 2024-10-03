---
title: "A Tutorial of Clickhouse"
subtitle: "A Tutorial of Clickhouse"
layout: post
author: "bulingfeng"
header-style: text
tags:
- ClickHouse
---

## What's the ClickHouse?

Clickhouse is a hight-performance, column-oriented SQL database management system for online analytical processing(OLAP).

### What's OLAP?

OLAP scenarios require real time responses on top of large datasets for complex analytical queries with following characteristics:

- Datasets can be massive - billions or trillions of rows
- Data is organized in tables that contain many columns
- Only a few columns are selected to answer any particular query
- Results must be returned in millionseconds or seconds

## Install ClickHouse

I install ClickHouse on my macbook, if you want to install it on your machine, please visit offical website for finding suitable package.

```
curl https://clickhouse.com/ | sh
./clickhouse
```

## Common SQL in ClickHouse

### Creating Tables in ClickHouse

```sql
CREATE TABLE helloworld.my_first_table
(
    user_id UInt32,
    message String,
    timestamp DateTime,
    metric Float32
)
ENGINE = MergeTree()
PRIMARY KEY (user_id, timestamp)
```

Primary Key is different from MYSQL, primary keys in ClickHouse are ***not unique\*** for each row in a table.

The primary key of a ClickHouse table determines how the data is sorted when written to disk. Every 8,192 rows or 10MB of data (referred to as the **index granularity**) creates an entry in the primary key index file. This granularity concept creates a **sparse index** that can easily fit in memory, and the granules represent a stripe of the smallest amount of column data that gets processed during `SELECT` queries.

## Inserting Data

```sql
INSERT INTO helloworld.my_first_table (user_id, message, timestamp, metric) VALUES
    (101, 'Hello, ClickHouse!',                                 now(),       -1.0    ),
    (102, 'Insert a lot of rows per batch',                     yesterday(), 1.41421 ),
    (102, 'Sort your data based on your commonly-used queries', today(),     2.718   ),
    (101, 'Granules are the smallest chunks of data read',      now() + 5,   3.14159 )
```

## Select Data

```sql
SELECT * FROM helloworld.my_first_table ORDER BY timestamp
```

## Updating and Deleting ClickHouse Data

### updating data

```sql
ALTER TABLE website.clicks
UPDATE url = substring(url, position(url, '://') + 3), visitor_id = new_visit_id
WHERE visit_date < '2022-01-01'
```

### deleting data

```sql
ALTER TABLE website.clicks DELETE WHERE visitor_id in (253, 1002, 4277)
```

### Lightweight Deletes

Another option for deleting rows it to use the `DELETE FROM` command, which is referred to as a **lightweight delete**. The deleted rows are marked as deleted immediately and will be automatically filtered out of all subsequent queries, so you do not have to wait for a merging of parts or use the `FINAL` keyword. Cleanup of data happens asynchronously in the background.

```sql
DELETE FROM hits WHERE Title LIKE '%hello%';
```

