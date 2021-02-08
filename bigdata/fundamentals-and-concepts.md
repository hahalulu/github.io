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
String representations of data:

require text encode/decode (which can be cheap, but is still an extra step)
requires complex parse code, especially if there are human-friendly rules like "must allow whitespace"
usually involves more bandwidth - so more actual payload to churn - due to embedding of things like names, and (again) having to deal with human-friendly representations (how to tokenize the syntax, for example)
often requires lots of intermediate string instances that are used for member-lookups etc
Both text-based and binary-based serializers can be fast and efficient (or slow and horrible)... just: binary serializers have the scales tipped in their advantage. This means that a "good" binary serializer will usually be faster than a "good" text-based serializer.

Let's compare a basic example of an integer:

*json:*
`{"id":42}`
9 bytes if we assume ASCII or UTF-8 encoding and no whitespace.
*xml:*

`<id>42</id>`
11 bytes if we assume ASCII or UTF-8 encoding and no whitespace - and no namespace noise like namespaces.

*protobuf:*

`0x08 0x2a`
2 bytes

Now imagine writing a general purpose xml or json parser, and all the ambiguities and scenarios you need to handle just at the text layer, then you need to map the text token "id" to a member, then you need to do an integer parse on "42". In protobuf, the payload is smaller, plus the math is simple, and the member-lookup is an integer (so: suitable for a very fast switch/jump).




### How bucketing reduces shuffling?

### Bloom filter and how it improves the performance

### what is write serializaion?
### what is Z


## Apache Hudi / Delta Lake / Apache Iceberg.
[Comparison of Apache Hudi vs Apache Iceberg vs Delta](https://eric-sun.medium.com/rescue-to-distributed-file-system-2dd8abd5d80d)
 - [Databricks analysis on the above (slides)](https://www.slideshare.net/databricks/a-thorough-comparison-of-delta-lake-iceberg-and-hudi)
 - [Databricks video and transcript](https://databricks.com/session_na20/a-thorough-comparison-of-delta-lake-iceberg-and-hudi)
 
### Features that we expect Data Lake's to have

1. Transaction or ACID ability.
2. Time travel, concurrence read, and write.
3. Support for batch and streaming
4. Since data is allowed to be late in streaming we need to have a mechanism like data mutation and data correction which would allow the right data to merge into the base dataset and the correct base dataset to follow for the business view of the report for end-user.
5. As the table made changes around with the business over time. So we also expect that data lake to have features like Schema Evolution and Schema Enforcements, which could update a Schema over time.
6. Data Lake is, independent of the engines and the underlying storages.

#### Delta Lake
- [How traansactions work in delta lake](https://databricks.com/blog/2019/08/21/diving-into-delta-lake-unpacking-the-transaction-log.html)
i. Dealing with Multiple Concurrent Reads and Writes, uses optimistic concurrency control.
ii. For Solving Conflicts Optimistically, uses  mutual exclusion.
- [How schema validation and schema evolution works](https://databricks.com/blog/2019/09/24/diving-into-delta-lake-schema-enforcement-evolution.html)
- [Update/Merge/Delete](https://databricks.com/blog/2020/09/29/diving-into-delta-lake-dml-internals-update-delete-merge.html)
  - Scan for records , select those files, update those file, old files are tombstoned.
  
#### Apache Hudi
(TODO)
#### Apache Iceberg
(TODO)

## Apache Kafka
[Apache Kafka Schema management](https://docs.confluent.io/platform/current/schema-registry/index.html#)







 

