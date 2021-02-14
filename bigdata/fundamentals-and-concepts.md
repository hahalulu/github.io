# Fundamentals and Concepts

## Data Encoding and Schema Evolution

1. [How Protobuf and Avro and Thrift allow schema evolution](https://martin.kleppmann.com/2012/12/05/schema-evolution-in-avro-protocol-buffers-thrift.html)
    - Avro, follows or forces you to follow stricter guidelines in terms of schema evolution.
    - Protobuf has lax conditions and it can allow schema evolution without enforcing any constraints
    so its upto users to make sure its followed.
    - Use Protobuf, if you have a single service and want to service data between them.
    - If there is no dependency on downstream to be on same version  
2. [How Protobuf actually encodes the data - Internal working](https://medium.com/@yashschandra/an-inner-view-to-protobuf-encoding-e668f37847d5#)

### Advantages of Protocol Buffers - Comparison with JSON and not avro

- String representations of data:
- Require text encode/decode (which can be cheap, but is still an extra step)
- Requires complex parse code, especially if there are human-friendly rules like "must allow whitespace"
- Usually involves more bandwidth - so more actual payload to churn - due to embedding of things like names,and (again) having to deal with human-friendly representations (how to tokenize the syntax, for example)
- Often requires lots of intermediate string instances that are used for member-lookups etc
- Both text-based and binary-based serializers can be fast and efficient (or slow and horrible)... just: binary serializers have the scales tipped in their advantage. This means that a "good" binary serializer will usually be faster than a "good" text-based serializer.

Let's compare a basic example of an integer:

**json:**
`{"id":42}`
9 bytes if we assume ASCII or UTF-8 encoding and no whitespace.

**xml:**
`<id>42</id>`
11 bytes if we assume ASCII or UTF-8 encoding and no whitespace - and no namespace noise like namespaces.

**protobuf:**
`0x08 0x2a`
2 bytes

Now imagine writing a general purpose xml or json parser, and all the ambiguities and scenarios you need to handle just at the text layer, then you need to map the text token "id" to a member, then you need to do an integer parse on "42". In protobuf, the payload is smaller, plus the math is simple, and the member-lookup is an integer (so: suitable for a very fast switch/jump).

### How bucketing reduces shuffling?


### Bloom filter and how it improves the performance
[What is bloom filter](http://hadoopnalgos.blogspot.com/2017/02/bloom-filter.html)
[Addvantages of bloom filter](https://stackoverflow.com/questions/4282375/what-is-the-advantage-to-using-bloom-filters)
TL;DR - Used to check if element is definitely not there.
[Online bloom filter playground][https://llimllib.github.io/bloomfilter-tutorial/
]
```text
A false positive means that the results say you have the condition you were tested for, 
but you really don't. With a false negative, the results say you don't have a condition, but you really do.
```


## Isolation levels
- Serializable: The strongest isolation level. It ensures that committed write operations and all reads are Serializable. Operations are allowed as long as there exists a serial sequence of executing them one-at-a-time that generates the same outcome as that seen in the table. For the write operations, the serial sequence is exactly the same as that seen in the table’s history.
- WriteSerializable: A weaker isolation level than Serializable. It ensures only that the write operations (that is, not reads) are serializable. However, this is still stronger than Snapshot isolation. WriteSerializable is the default isolation level because it provides great balance of data consistency and availability for most common operations.
- Snapshot serialization: In databases, and transaction processing (transaction management), snapshot isolation is a guarantee that all reads made in a transaction will see a consistent snapshot of the database (in practice it reads the last committed values that existed at the time it started), and the transaction itself will successfully commit only if no updates it has made conflict with any concurrent updates made since that snapshot.
[Isolation levels](https://docs.databricks.com/delta/optimizations/isolation-level.html)
    
### Data Skipping and Z-ordering
- [Data Skipping and Z-ordering in Delta lake](https://engineering.salesforce.com/boost-delta-lake-performance-with-data-skipping-and-z-order-75c7e6c59133)
- [Databricks optimization technique](https://docs.databricks.com/delta/optimizations/file-mgmt.html#compaction-bin-packing)

    

## Apache Hudi / Delta Lake / Apache Iceberg.
[Comparison of Apache Hudi vs Apache Iceberg vs Delta](https://eric-sun.medium.com/rescue-to-distributed-file-system-2dd8abd5d80d)
 - [Databricks analysis on the above (slides)](https://www.slideshare.net/databricks/a-thorough-comparison-of-delta-lake-iceberg-and-hudi)
 - [Databricks video and transcript](https://databricks.com/session_na20/a-thorough-comparison-of-delta-lake-iceberg-and-hudi)
 
### Features that we expect Data Lake's to have

1. Transaction or ACID ability.
2. Time travel, concurrence read, and write.
3. Support for both batch and streaming
4. data could be late in streaming, thus we need to have a mechanism like data mutation and data correction which would allow the right data to merge into the base dataset, and the correct base dataset to follow for the business view of the report for end-user.
5. As the table made changes around with the business over time. So we also expect that data lake to have features like Schema Evolution and Schema Enforcements, which could update a Schema over time.
6. Data Lake is, independent of the engines and the underlying storage's.

#### Delta Lake
- [How traansactions work in delta lake](https://databricks.com/blog/2019/08/21/diving-into-delta-lake-unpacking-the-transaction-log.html)
i. Dealing with Multiple Concurrent Reads and Writes, uses optimistic concurrency control.
ii. For Solving Conflicts Optimistically, uses  mutual exclusion.
- [How schema validation and schema evolution works](https://databricks.com/blog/2019/09/24/diving-into-delta-lake-schema-enforcement-evolution.html)
- [Update/Merge/Delete](https://databricks.com/blog/2020/09/29/diving-into-delta-lake-dml-internals-update-delete-merge.html)
  - Scan for records , select those files, update those file, old files are tombstone.
  - TODO : need to understand all performance tuning in this case.
  
- [Optimize File Management (compaction)](https://docs.databricks.com/delta/optimizations/file-mgmt.html#compaction-bin-packing)
- [Data Skipping and Z-ordering](https://engineering.salesforce.com/boost-delta-lake-performance-with-data-skipping-and-z-order-75c7e6c59133)

  
#### Apache Hudi
(TODO)
#### Apache Iceberg
(TODO)

### Natural Keys vs Synthetic Keys.


## Apache Kafka
[Basics of kafka ecosystem]()

[Apache Kafka Schema management](https://docs.confluent.io/platform/current/schema-registry/index.html#)
[Kafka avro vs Kafka proto](https://simon-aubury.medium.com/kafka-with-avro-vs-kafka-with-protobuf-vs-kafka-with-json-schema-667494cbb2af)

[Kafka Exactly once (Effectively once), Atleast once](https://medium.com/@andy.bryant/processing-guarantees-in-kafka-12dd2e30be0e)
[Original confluent blog for kafka exactly once](https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/)

[How to achieve strict ordering in kafka](https://www.cloudkarafka.com/blog/2018-08-21-faq-apache-kafka-strict-ordering.html)

[Read topic from beginning](https://riptutorial.com/apache-kafka/example/19390/how-can-i-read-topic-from-its-beginning)

[Kafka storage internals](https://medium.com/@durgaswaroop/a-practical-introduction-to-kafka-storage-internals-d5b544f6925f#:~:text=To%20confirm%20that%20the%20messages,what's%20inside%20that%20log%20file.&text=Kafka%20stores%20all%20the%20messages,also%20called%20as%20the%20Offset%20.)

[Disaster recovery for kafka multi-region uber](https://eng.uber.com/kafka/) todo

[Read data from closest replica for reducing the latency](https://developers.redhat.com/blog/2020/04/29/consuming-messages-from-closest-replicas-in-apache-kafka-2-4-0-and-amq-streams/)
[Proposal for closest replica changes](https://cwiki.apache.org/confluence/display/KAFKA/KIP-392%3A+Allow+consumers+to+fetch+from+closest+replica)


## Apache Kinesis
[Medium blog on Kinesis](https://medium.com/@yashbindlish1/amazon-kinesis-the-core-of-real-time-streaming-a543085a212f)


### Kinesis vs Kafka
[Benchmarking Kinesis vs Kafka](https://medium.com/flo-engineering/kinesis-vs-kafka-6709c968813)
[Kinesis vs Kafka Analysis](https://medium.com/softkraft/aws-kinesis-vs-kafka-comparison-which-is-right-for-you-8e81374d8166)


### What is compaction 
[Compaction in HDFS](https://medium.com/datakaresolutions/compaction-in-hive-97a1d072400f)

## Good reads
[Why GCP is better than AWS?](https://nandovillalba.medium.com/why-i-think-gcp-is-better-than-aws-ea78f9975bda)

## Data modelling 
[Parent child relationship in data modelling using Avro](https://www.linkedin.com/pulse/parent-child-relationships-you-joshua-hartman/)

[Avro Schema Composition](https://mykidong.medium.com/howto-implement-avro-schema-inheritance-757d2897c1ad)

[Data Governance using Avro and Kafka](https://medium.com/@sderosiaux/governing-data-with-kafka-avro-ecfb665f266c)


## Avro vs Parquet
###COMPARISONS BETWEEN DIFFERENT FILE FORMATS
**AVRO vs PARQUET**
- AVRO is a row-based storage format whereas PARQUET is a columnar based storage format.
- PARQUET is much better for analytical querying i.e. reads and querying are much more efficient than writing.
- Write operations in AVRO are better than in PARQUET.
- AVRO is much matured than PARQUET when it comes to schema evolution. PARQUET only supports schema append whereas AVRO supports a much-featured schema evolution i.e. adding or modifying columns.
- PARQUET is ideal for querying a subset of columns in a multi-column table. AVRO is ideal in case of ETL operations where we need to query all the columns.
**ORC vs PARQUET**
- PARQUET is more capable of storing nested data.
- ORC is more capable of Predicate Pushdown.
- ORC supports ACID properties.
- ORC is more compression efficient.


[Introduction to columnar formats in spark and hadoop](https://blog.matthewrathbone.com/2019/11/21/guide-to-columnar-file-formats.html)
[Intro to hive file format (paid medium)](https://towardsdatascience.com/new-in-hadoop-you-should-know-the-various-file-format-in-hadoop-4fcdfa25d42b)
[Simple high level diff - Avro,ORC,Parquet](https://towardsdatascience.com/demystify-hadoop-data-formats-avro-orc-and-parquet-e428709cf3bb)
[Medium level explanation of all formats](https://towardsdatascience.com/new-in-hadoop-you-should-know-the-various-file-format-in-hadoop-4fcdfa25d42b)
[Medium level explanation of different file formats](https://blog.clairvoyantsoft.com/big-data-file-formats-3fb659903271)

[ORC performance and compression > parquet](https://blog.cloudera.com/orcfile-in-hdp-2-better-compression-better-performance/)

- Spark's optimized for parquet, hive's optimized for orc file format.
- Parquet is implemented using the google dremel paper. Hence, parquet is intended for complex data structure types.(tree like)
- Many of the performance improvements provided in the Stinger initiative are dependent on features of the ORC format including block level index for each column.
  This leads to potentially more efficient I/O allowing Hive to skip reading entire blocks of data if it determines predicate values are not present there.
  Also the Cost Based Optimizer has the ability to consider column level metadata present in ORC files in order to generate the most efficient graph.
[Hive 3.0 ACID compliance only with ORC](https://docs.cloudera.com/HDPDocuments/HDP3/HDP-3.1.5/using-hiveql/content/hive_hive_3_tables.html)

[How compression effects file splitting](http://comphadoop.weebly.com/)

### Avro examples
    
### Avro tools and code

### Proto examples 

### Proto tools and code
    

 

