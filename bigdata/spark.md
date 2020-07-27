
# Spark


## Spark: What’s Underneath an RDD?


## What is the difference between Spark - repartition() vs coalesce()

Please check the below link to understand the difference
[spark repartition vs coalesce](https://stackoverflow.com/questions/31610971/spark-repartition-vs-coalesce)


### why the size of a data lake increases after aggregations are ran?

In general, columnar storage formats like Parquet are highly
sensitive when it comes to data distribution (data organization) and 
cardinality of individual columns. The more organized the data and lower 
the cardinality the more efficient is the storage.
**Cardinality**
- the term cardinality refers to the uniqueness of data values contained in a particular column (attribute) of a database table.
 The lower the cardinality, the more duplicated elements in a column.
 
 
 
what does bucket by do?




### Different type of joins supported by spark
Spark has five distinct join strategies by which it exchanges, moves, sorts, groups, and merges data across executors:
- Broadcast hash join (BHJ)
- Shuffle hash join (SHJ)
- Shuffle sort merge join (SMJ)
- Broadcast nested loop join (BNLJ)
- Shuffle-and-replicated nested loop join (a.k.a. Cartesian product join).
 We’ll focus on only two of these here (BHJ and SMJ), because they’re the most common ones you’ll encounter.
 
