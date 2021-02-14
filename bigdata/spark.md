
1.x - RDD framework
2.x - DataSets / Dataframe
    

### Important Spark Tutorials and Blog
[Important spark links - D-Zone](https://dzone.com/articles/the-complete-apache-spark-collection-tutorials-and)

## re-partition vs coalesce 
```text
Keep in mind that repartitioning your data is a fairly expensive operation. 
Spark also has an optimized version of repartition() called coalesce() that allows avoiding data movement,
 but only if you are decreasing the number of RDD partitions.
```
ex:

```text

It avoids a full shuffle. If it's known that the number is decreasing then the executor can safely keep data on the minimum number of partitions, only moving the data off the extra nodes, onto the nodes that we kept.

So, it would go something like this:

Node 1 = 1,2,3
Node 2 = 4,5,6
Node 3 = 7,8,9
Node 4 = 10,11,12
Then coalesce down to 2 partitions:

Node 1 = 1,2,3 + (10,11,12)
Node 3 = 7,8,9 + (4,5,6)
Notice that Node 1 and Node 3 did not require its original data to move.
```

**Difference between coalesce and repartition**

coalesce uses existing partitions to minimize the amount of data that's shuffled. repartition creates new partitions and does a full shuffle. coalesce results in partitions with different amounts of data (sometimes partitions that have much different sizes) and repartition results in roughly equal sized partitions.


### Spark SQL joins & performance tuning
[Join strategies Broadcast Hash Join, Shuffle Hash Join, Shuffle Sort Merge, Iterative Broadcast Join](https://towardsdatascience.com/strategies-of-spark-join-c0e7b4572bcf)

*Troubleshooting shuffle / uneven sharding - Some task are taking a lot of time. speculative task's are triggered.*

**ShuffleHashJoin** (Impt)
 - The join keys don’t need to be sortable.
 - Supported for all join types except full outer joins.
 
Problems with shuffle joins usually are 
 - it’s an expensive join in a way that involves both shuffling and hashing(Hash Join as explained above). Maintaining a hash table requires memory and computation.
 - uneven sharding and limited parallelism for instance if we perform a query on US census and try to (inner) join on states
 what is going to US DF is very big and state is going to be very small so the job the bottlenecked on only a certain
 state where the data points are lot and significantly less resource will be used for smaller states.
 
 ```python
    join_df = sqlContext.sql("Select * FROM people_in_the_US JOIN states ON people_in_the_US.states = states");
```
 ```python
    sqlContext.sql("Select * FROM people_in_cali  LEFT JOIN all_people_in_world ON people_in_cali.id = all_people_in_world.id");
```

**BroadcastHashJoin** (Impt)
 - If one dataframe is small and could be fit in memory we simply put it in memory and send it to all the worker nodes.
 - It doesn't require you to do any shuffle as the operations are performed on worker nodes.
 - Use explain command to see which JOIN was selected by sql catalyst.
 - Spark deploys this join strategy when the size of one of the join relations is less than the threshold values(default 10 M). The spark property which defines this threshold is spark.sql.autoBroadcastJoinThreshold(configurable).
 
**SortMergeJoin**

 
**CartesianJoin**
Not much explained here.

**One to Many Joins**
One to many joins - parquet took care of it?

**Theta Joins**
Join is based on a condition - Full cartesian join and then do the condition.
Generate bucket to match condition and match the buckets?? not clear about this as well.


### JOIN
`join(self, other, on=None, how=None)`

how=left,right,inner,outer
ex : `empDF.join(deptDF,empDF.emp_dept_id ==  deptDF.dept_id, "inner")`


## Tackling Data Skewness in Spark
- [Iterative Broadcaast Join optimization](https://github.com/godatadriven/iterative-broadcast-join)
- [Data Skewness](https://bigdatacraziness.wordpress.com/2018/01/05/oh-my-god-is-my-data-skewed/)
- [Salting code in Python to reduce data skew](https://datarus.wordpress.com/2015/05/04/fighting-the-skew-in-spark/)
- [Trouble shooting data skewness](https://dzone.com/articles/why-your-spark-apps-are-slow-or-failing-part-ii-da)


## Tackling memory issues in spark
- [Troubleshooting memory issues in spark](https://dzone.com/articles/common-reasons-your-spark-applications-are-slow-or)

## Spark Optimization
- [Spark Core - Proper Optimization](https://www.youtube.com/watch?v=daXEp4HmS-E&ab_channel=Databricks) (TODO)


## GroupBy vs ReduceBy
[Stack Overflow - differences](https://medium.com/@sderosiaux/governing-data-with-kafka-avro-ecfb665f266c)
```text

Group by Key

sparkContext.textFile("s3://../..")
			.flatMap(lambda line: line.split())
			.map(lambda word: (word,1))
			.groupByKey()
			map(lambda (w,counts) : (w, sum(counts)))


Reduce By Key

spaarkContext.textFile("s3://../..")
			.flatMap(lambda line: line.split())
			.map(lambda word: (word,1))
			reduceByKey(lambda a,b : a+b)


Reduce by key will perform better because it will combine the results in the node before sending it over.
whereas GBK all the records will have to be moved to
its respective partitions in appropriate nodes.


We can not always apply ReduceByKey as ReduceByKey requires combining all the values into another value 
with the exact same type.

aggregateByKey, foldByKey, combineByKey
```

## Spark Cluster Management
[Deep Dive](https://dzone.com/articles/deep-dive-into-spark-cluster-management)


## Ideal Paritition count 
The recommended number of partitions is around 3 or 4 times the number of CPUs in the cluster so that the work gets distributed more evenly among the CPUs.

### What is synthetic keys in hive? What are they used for?


## Data Modelling interview questions
[Interview questions](https://www.guru99.com/data-modeling-interview-questions-answers.html)


### Defining schema in spark 

```sparksql
from pyspark.sql.types import
schema = StructType([StructField("author", StringType(), False),  StructField("title", StringType(), False),  StructField("pages", IntegerType(), False)])
```

### Create a SparkSession
```sparksql
   spark = (SparkSession
     .builder
     .appName("Example-3_6")
     .getOrCreate())
```
### Read data in DF
```sparksql
file = "/databricks-datasets/learning-spark-v2/flights/summary-data/orc/*"
df = spark.read.format("orc").option("path", file).load()
df.show(10, False)
```
### Read binary file in DF
```sparksql
path = "/databricks-datasets/learning-spark-v2/cctvVideos/train_images/"
binary_files_df = (spark.read.format("binaryFile")
  .option("pathGlobFilter", "*.jpg")
  .load(path))
binary_files_df.show(5)
```




























































