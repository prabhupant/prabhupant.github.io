---
layout: post
title: "Joins in Databases"
---

Yesterday while writing SQL queries, I question came into my mind — how do SQL joins work internally? I had no idea and started to figure it out using first principles — given two lists of say, integers, how would I find intersections?
The naive way is to write a nested for loop, compare all elements and then store the result in a new array. Can we improve on this? Combine arrays and use merge sorts? Now think of a database. Databases have an index so how can we leverage that? Basically, our join operation should be like — find row_table1 where row_table1.id = row_table2.id. Since it's a find operation, using something like a hash map can help. But unlike a cache, databases are way too big to store in memory. So the database engine should do some query planning and optimisation to figure out this out.
Let’s see. In this case I am considering Postgres as my database.

## Types of joins

There are three types of joins that DB uses internally —

### Nested Loop Join

Like our naive idea, Nested Loop Join uses two for loops to find the intersecting rows and fetch them for the join operation. In this query, the DB engine asks for all rows in both the tables, does a sequential scan and combine the result into the desired output format. Thus this has no room for optimisation and for large queries where tables have indexes, this type of join does not scale well.

But there is are two scenarios where this is useful and DB engine uses this join instead of the other two

1. **One of the table has few rows:** in this case, the outer for loop will be much smaller than the inner for loop. Hence the computation cost of this operation would be much lower. Also you might ask, which table is outer and which is inner? It really depends on the databases and most of them select the smaller table as the outer table.
2. **When the join condition has no equality operator:** In this case, you can’t leverage hash tables’ lookup operation so you need to compare every element of both the tables

### Hash Join

Hash joins are preferred if there is an equality operator in the join operation and both sides of the join are large enough to fit into the work_mem, which defines the maximum memory to be used for query workspaces ([source](https://postgresqlco.nf/doc/en/param/work_mem/)).

This join is performed by hashing one data set into memory by join column and reading the other one by one, taking its join column’s hash and checking if it exists in the database.

The cost of this rises if the table is too big and it has to be split into multiple parts so as to fit into the work_mem. The cost of a hash join can be reduced by partitioning both tables on the join key. This allows the optimiser to infer that rows from a partition in one table will only find a match in a particular partition of the other table, and for tables having n partitions the hash join is executed as n independent hash joins

### Merge Join

Merge joins are more flexible than hash joins in that you don’t require equality operator to always exist. Like merge sort, this algorithm sorts the dataset and then merges them according to the join key.

Making your index the join key helps a lot to increase its performance. This type of join is particularly useful when the dataset is way too big to fit into work_mem

---

Helpful sources —

1. https://stackoverflow.com/questions/1111707/what-is-the-difference-between-a-hash-join-and-a-merge-join-oracle-rdbms
2. This article covers the query plan as well — https://severalnines.com/blog/overview-join-methods-postgresql/
3. https://www.cybertec-postgresql.com/en/join-strategies-and-performance-in-postgresql/
