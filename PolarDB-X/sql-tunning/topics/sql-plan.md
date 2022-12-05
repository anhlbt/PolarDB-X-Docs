Implementation Plan Introduction
===========================

This article introduces how to use query execution plans and introduces some basic operator meanings and implementations.

Operator introduction
-------------------------



| Meaning | Physical operator |
|---------------------------------|-------------------------------------------------------------------------------------------|
| Operators delivered to DN | LogicalView, LogicalModifyView, PhyTableOperation, IndexScan |
|电视（Join） | BKAJoin，NLJoin，HashJoin，SortMergeJoin，HashSemiJoin，SortMergeSemiJoin，MaterializedSemiJoin |
| Sorting | MemSort, TopN, MergeSort |
| Aggregation (Group By) | HashAgg, SortAgg |
| Data redistribution or aggregation | Exchange Gather |
| Filter | Filter |
| Projection | Project |
| Union | Union |
| Set the number of output rows in the result set (Limit/Offset...Fetch) | Limit |
| Window Functions | OverWindow |



**LogicalView**

LogicalView is an operator that pulls data from MySQL data sources in the storage layer, similar to TableScan or IndexScan in other databases, but supports more pushdowns. LogicalView contains pushed-down SQL statements and data source information, more like a view. The pushed-down SQL may contain various operators, such as Project, Filter, Aggregation, Sorting, Join, and subqueries. The following example shows you the output information of LogicalView in EXPLAIN and its meaning:

```sql
mysql> explain select * From sbtest1 where id > 1000;
Gather(concurrent=true)
LogicalView(tables="[0000-0031].sbtest1_[000-127]", shardCount=128, sql="SELECT * FROM `sbtest1` WHERE (`id` > ?)")
```

LogicalView's information consists of three parts:

* tables: The table name corresponding to the storage layer MySQL, separated by English period (.), before the English period (.) is the number corresponding to the sub-database, followed by the table name and its number, such as [000-127] indicates the number of the table name All tables from 000 to 127.

* shardCount: The total number of shards to be accessed, this example will access a total of 128 shards from 000 to 127.

* sql: The SQL template sent to the storage layer MySQL. PolarDB-X will replace the table name with the physical table name during execution, and replace the parameterized constant question mark (?) with the actual parameter. For details, please refer to [Execution Plan Management] (spm.md).




**IndexScan**

IndexScan, like LogicalView, is also an operator that pulls data from the storage layer MySQL data source, and scans the index table. The following example shows you the output information of IndexScan in EXPLAIN and its meaning:

```sql
mysql> explain select * from sequence_one_base where integer_test=1;
|
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| IndexScan(tables="DRDS_POLARX1_QATEST_APP_000000_GROUP.gsi_sequence_one_index_3a0A_01", sql="SELECT `pk`, `integer_test`, `varchar_test`, `char_test`, `blob_test`, `tinyint_test`, `tinyint_1bit_test`, `smallint_test`, `mediumint_test`, `bit_test`, `bigint_test`, `float_test`, `double_test`, `decimal_test`, `date_test`, `time_test`, `datetime_test`, `timestamp_test`, `year_test`, `mediumtext_test` FROM `gsi_dml_sequence_one_index_index1` AS `gsi_dml_sequence_one_index_index1` WHERE (`integer_test` = ?)") |
```

As a predicate condition, the index table is trimmed, so in fact, we will generate the IndexScan operator, which means that only one slice of the gsi_sequence_one_index index table will be scanned. The above SQL should normally scan the sequence_one_base table. Since integer_test is not a partition key, all fragments of sequence_one_base need to be scanned. However, since the sequence_one_base table has a global secondary index gsi_sequence_one_index on the integer_test column, the

**Gather**

Gather merges multiple data into the same data. In the above example, Gather merges the data queried on each sub-table into one. Gather usually appears above the LogicalView, which means collecting and merging the data of each sub-table.

**Exchange**

Exchange is a logical operator that does not perform calculations on the data in the calculation process, but redistributes the input data and outputs it to downstream operators. The general redistribution strategy is divided into

* SINGLETON: Merge and output multiple upstream data, this redistribution strategy is equivalent to Gather

* HASH_DISTRIBUTED: Repartition the upstream input data according to certain columns, which is common in execution plans containing Join and Agg.

* BROADCAST_DISTRIBUTED: Distribute the same upstream data into multiple copies and broadcast to multiple downstream nodes, mainly used in MPP execution plan.




**MergeSort**

MergeSort is a merge sort operator, which means to merge and sort ordered data streams and merge them into an ordered data stream. E.g:

```sql
mysql> explain select * from sbtest1 where id > 1000 order by id limit 5,10;
MergeSort(sort="id ASC", offset=?1, fetch=?2)
LogicalView(tables="[0000-0031].sbtest1_[000-127]", shardCount=128, sql="SELECT * FROM `sbtest1` WHERE (`id` > ?) ORDER BY `id` LIMIT (? + ?)")
```

The MergeSort operator consists of three parts:

* sort: Indicates the sorting field and the order of sorting, id ASC indicates ascending sorting according to the ID field, and DESC indicates descending sorting.

* offset: Indicates the offset when obtaining the result set, which is parameterized in the example, and the actual value is 5.

* fetch: Indicates the maximum number of data rows returned. Similar to offset, it is also a parameterized representation, and the actual corresponding value is 10.




**Project**

Project represents a projection operation, that is, selects some columns to output from the input data, or converts some columns (calculated by functions or expressions) and outputs them. Of course, it can also contain constants.

```sql
mysql> explain select 'Hello, DRDS', 1 / 2, CURTIME();
Project(Hello, DRDS="_UTF-16'Hello, DRDS'", 1 / 2="1 / 2", CURTIME()="CURTIME()")
```

The project plan includes the column name of each column and its corresponding column, value, function or expression.

**Filter**

Filter represents a filtering operation, which contains some filtering conditions. This operator filters the input data, and if the condition is met, it is output, otherwise it is discarded. The following is a more complex example that includes most of the operators introduced above.

```sql
mysql> explain select k, avg(id) avg_id from sbtest1 where id > 1000 group by k having avg_id > 1300;
Filter(condition="avg_id > ?1")
Project(k="k", avg_id="sum_pushed_sum / sum_pushed_count")
SortAgg(group="k", sum_pushed_sum="SUM(pushed_sum)", sum_pushed_count="SUM(pushed_count)")
MergeSort(sort="k ASC")
LogicalView(tables="[0000-0031].sbtest1_[000-127]", shardCount=128, sql="SELECT `k`, SUM(`id`) AS `pushed_sum`, COUNT(`id`) AS `pushed_count` FROM `sbtest1` WHERE (`id` > ?) GROUP BY `k` ORDER BY `k`")
```

The condition in WHERE id > 1000 does not have a corresponding Filter operator, because this operator is finally pushed down to LogicalView, and you can see WHERE (id > ?) in the SQL of LogicalView.

**Union All与Union Distinct**

As the name implies, Union All corresponds to UNIONALL, and Union Distinct corresponds to UNIONDISTINCT. This operator usually has 2 or more inputs, which means combining data from multiple inputs. E.g:

```sql
mysql> explain select * From sbtest1 where id > 1000 union distinct select * From sbtest1 where id < 200;
UnionDistinct(concurrent=true)
Gather(concurrent=true)
LogicalView(tables="[0000-0031].sbtest1_[000-127]", shardCount=128, sql="SELECT * FROM `sbtest1` WHERE (`id` > ?)")
Gather(concurrent=true)
LogicalView(tables="[0000-0031].sbtest1_[000-127]", shardCount=128, sql="SELECT * FROM `sbtest1` WHERE (`id` < ?)")
```



**LogicalModifyView**

LogicalView represents the operator for obtaining data from the underlying data source. Correspondingly, LogicalModifyView represents the modification operator for the underlying data source, which also records a SQL statement, which may be INSERT, UPDATE, or DELETE.

```sql
mysql> explain update sbtest1 set c='Hello, DRDS' where id > 1000;
LogicalModifyView(tables="[0000-0031].sbtest1_[000-127]", shardCount=128, sql="UPDATE `sbtest1` SET `c` = ? WHERE (`id` > ?)"
```



```sql
mysql> explain delete from sbtest1 where id > 1000;
LogicalModifyView(tables="[0000-0031].sbtest1_[000-127]", shardCount=128, sql="DELETE FROM `sbtest1` WHERE (`id` > ?)")
```

The content of the LogicalModifyView query plan is similar to that of LogicalView, including the distributed physical sub-table, the number of sub-tables, and the SQL template. Similarly, since the execution plan cache is enabled and the SQL is parameterized, the constants in the SQL template will be replaced with ?.

**PhyTableOperation**

PhyTableOperation means to directly perform an operation on a physical sub-table.

**Note** Normally, this operator is only used in INSERT statements. But when the routing distribution is divided into a shard, the operator will also appear in the SELECT statement.

```sql
mysql> explain insert into sbtest1 values(1, 1, '1', '1'),(2, 2, '2', '2');
PhyTableOperation(tables="SYSBENCH_CORONADB_1526954857179TGMMSYSBENCH_CORONADB_VGOC_0000_RDS.[sbtest1_001]", sql="INSERT INTO ? (`id`, `k`, `c`, `pad`) VALUES(?, ?, ?, ?)", params="`sbtest1_001`,1,1,1,1")
PhyTableOperation(tables="SYSBENCH_CORONADB_1526954857179TGMMSYSBENCH_CORONADB_VGOC_0000_RDS.[sbtest1_002]", sql="INSERT INTO ? (`id`, `k`, `c`, `pad`) VALUES(?, ?, ?, ?)", params="`sbtest1_002`,2,2,2,2")
```

In the example, INSERT inserts two rows of data, and each row of data corresponds to a PhyTableOperation operator. The content of the PhyTableOperation operator consists of three parts:

* tables: Physical table name, only one physical table name.

* sql: SQL template, the table name and constants in this SQL template are parameterized, replaced by ?, and the corresponding parameters are given in the subsequent params.

* params: The parameters corresponding to the SQL template, including table names and constants.




Implementation Plan Introduction
---------------------------

After a piece of SQL enters the PolarDB-X distributed database, it will generate a runnable execution plan after parsing and optimization. The execution plan is composed according to the dependencies during operator execution. Generally, through the execution plan, you can spy on how efficiently SQL runs inside the database. For ease of understanding, here are a few examples.

**Example 1**



```sql
mysql> explain select count(*) from lineitem group by L_LINESTATUS;
|   HashAgg(group="L_LINESTATUS", count(*)="SUM(count(*))")                                                                                                                              |
|     Exchange(distribution=hash[0], collation=[])                                                                                                                                       |
|       LogicalView(tables="[000000-000003].lineitem_[00-15]", shardCount=16, sql="SELECT `L_LINESTATUS`, COUNT(*) AS `count(*)` FROM `lineitem` AS `lineitem` GROUP BY `L_LINESTATUS`")
```

Exchange: Summarize the data returned by LogicalView, redistribute it according to the L_LINESTATUS field, and output it to downstream operators; because the columns of the group by are not aligned with the partition key of the table lineitem, the group by cannot be completely pushed down to the DN for execution. Therefore, the group by will be split into two stages, and the partition agg will be pushed down to the DN to perform partial aggregation first; then the data will be redistributed at the CN layer, and then the final aggregation will be performed to output the result.

**Example 2**



```sql
mysql> explain select * from lineitem, orders where L_ORDERKEY= O_ORDERKEY;
|
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| HashJoin(condition="O_ORDERKEY = L_ORDERKEY", type="inner")                                                                                                                                                                                                                                                                                                      |
|   Exchange(distribution=hash[0], collation=[])                                                                                                                                                                                                                                                                                                                   |
|     LogicalView(tables="[000000-000003].lineitem_[00-15]", shardCount=16, sql="SELECT `L_ORDERKEY`, `L_PARTKEY`, `L_SUPPKEY`, `L_LINENUMBER`, `L_QUANTITY`, `L_EXTENDEDPRICE`, `L_DISCOUNT`, `L_TAX`, `L_RETURNFLAG`, `L_LINESTATUS`, `L_SHIPDATE`, `L_COMMITDATE`, `L_RECEIPTDATE`, `L_SHIPINSTRUCT`, `L_SHIPMODE`, `L_COMMENT` FROM `lineitem` AS `lineitem`") |
|   Exchange(distribution=hash[0], collation=[])                                                                                                                                                                                                                                                                                                                   |
|     LogicalView(tables="[000000-000003].orders_[00-15]", shardCount=16, sql="SELECT `O_ORDERKEY`, `O_CUSTKEY`, `O_ORDERSTATUS`, `O_TOTALPRICE`, `O_ORDERDATE`, `O_ORDERPRIORITY`, `O_CLERK`, `O_SHIPPRIORITY`, `O_COMMENT` FROM `orders` AS `orders`")
```

Example 2 is a typical join between two tables. Since the partition keys of the two tables are not aligned, all joins are not pushed down. The entire execution is to scan out the data of the two tables and perform the join calculation at the CN layer.

* LogicalView: Scan table data.

* Exchange: Summarize the data returned by LogicalView, distribute the data according to the columns of the associated conditions, and output it to the downstream Join operator.

* HashJoin: Accept input from both sides, and calculate the association result by building a HashTable.




**Example 3**



```sql
mysql> explain select * from lineitem, orders where L_LINENUMBER= O_ORDERKEY;
| Gather(concurrent=true)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|   LogicalView(tables="[000000-000003].lineitem_[00-15],orders_[00-15]", shardCount=16, sql="SELECT `lineitem`.`L_ORDERKEY`, `lineitem`.`L_PARTKEY`, `lineitem`.`L_SUPPKEY`, `lineitem`.`L_LINENUMBER`, `lineitem`.`L_QUANTITY`, `lineitem`.`L_EXTENDEDPRICE`, `lineitem`.`L_DISCOUNT`, `lineitem`.`L_TAX`, `lineitem`.`L_RETURNFLAG`, `lineitem`.`L_LINESTATUS`, `lineitem`.`L_SHIPDATE`, `lineitem`.`L_COMMITDATE`, `lineitem`.`L_RECEIPTDATE`, `lineitem`.`L_SHIPINSTRUCT`, `lineitem`.`L_SHIPMODE`, `lineitem`.`L_COMMENT`, `orders`.`O_ORDERKEY`, `orders`.`O_CUSTKEY`, `orders`.`O_ORDERSTATUS`, `orders`.`O_TOTALPRICE`, `orders`.`O_ORDERDATE`, `orders`.`O_ORDERPRIORITY`, `orders`.`O_CLERK`, `orders`.`O_SHIPPRIORITY`, `orders`.`O_COMMENT` FROM `lineitem` AS `lineitem` INNER JOIN `orders` AS `orders` ON (`lineitem`.`L_LINENUMBER` = `orders`.`O_ORDERKEY`)") |
```

Example 3 is also a typical association between two tables. Since the partition keys of the two tables are aligned, all joins are pushed down to the DN of each shard for execution. The upper-level CN ​​node only needs to summarize and output the results returned by the DN through the Gather operator.

**Example 4**



```sql
mysql> explain select * from gsi_dml_unique_multi_index_base where integer_test=1;                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Project(pk="pk", integer_test="integer_test", varchar_test="varchar_test", char_test="char_test", blob_test="blob_test", tinyint_test="tinyint_test", tinyint_1bit_test="tinyint_1bit_test", smallint_test="smallint_test", mediumint_test="mediumint_test", bit_test="bit_test", bigint_test="bigint_test", float_test="float_test", double_test="double_test", decimal_test="decimal_test", date_test="date_test", time_test="time_test", datetime_test="datetime_test", timestamp_test="timestamp_test", year_test="year_test", mediumtext_test="mediumtext_test") |
|   BKAJoin(condition="pk = pk", type="inner")                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
|     IndexScan(tables="DRDS_POLARX1_QATEST_APP_000000_GROUP.gsi_dml_unique_multi_index_index1_a0ol_01", sql="SELECT `pk`, `integer_test`, `varchar_test`, `char_test`, `bit_test`, `bigint_test`, `double_test`, `date_test` FROM `gsi_dml_unique_multi_index_index1` AS `gsi_dml_unique_multi_index_index1` WHERE (`integer_test` = ?)")                                                                                                                                                                                                                              |
|     Gather(concurrent=true)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
|       LogicalView(tables="[000000-000003].gsi_dml_unique_multi_index_base_[00-15]", shardCount=16, sql="SELECT `pk`, `blob_test`, `tinyint_test`, `tinyint_1bit_test`, `smallint_test`, `mediumint_test`, `float_test`, `decimal_test`, `time_test`, `datetime_test`, `timestamp_test`, `year_test`, `mediumtext_test` FROM `gsi_dml_unique_multi_index_base` AS `gsi_dml_unique_multi_index_base` WHERE ((`integer_test` = ?) AND (`pk` IN (...)))")                                                                                                                 |
| HitCache:true
```

This example is very interesting. SQL itself is just a simple query with predicates. From the execution plan, the result is an association between two tables (BKAJoin). The main reason is that gsi_dml_unique_multi_index_base has a global secondary index on the integer_test column. Hitting the index can reduce the scanning cost, but this index is not a covering index, so a table return operation is required.

* IndexScan: Scan the index table gsi_dml_unique_multi_index_index1_a0ol_01 data according to integer_test=1.

* BKAJoin: Collect the results of IndexScan, and associate the operator with the main table gsi_dml_unique_multi_index_base to obtain other column values.


For more information about BKAJoin, please refer to ["How to Implement Join in Distributed Databases"](https://zhuanlan.zhihu.com/p/363151441).

**Note** Usually, by querying the execution plan, you can check whether the global secondary index is hit or not. But for the pushed-down part of the SQL, you can also use the explain execute command to obtain the execution status of the physical SQL on the DN, such as whether the local index of the DN is hit.
