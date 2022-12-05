Tuning Methodology
==========================

After finding the slow SQL that needs to be tuned, first check the execution plan through EXPLAIN, and then optimize the SQL through the following methods: push down more calculations to the storage layer MySQL, increase indexes appropriately, and optimize the execution plan.

push down more calculations
----------------------------

PolarDB-X pushes down as many calculations as possible to the storage layer MySQL. Pushdown computing can reduce data transmission, reduce the overhead of the network layer and PolarDB-X layer, and improve the execution efficiency of SQL statements. PolarDB-X supports pushdown of almost all operators, including:

* Filter conditions, such as conditions in WHERE or HAVING.

* Aggregation operators, such as COUNT, GROUP BY, etc., will be divided into two stages for aggregation calculation.

* Sorting operator, such as ORDER BY.

* For JOIN and subquery, both sides of the JOIN Key must be fragmented in the same way, or one of them must be a broadcast table.


The following example explains how to push down more calculations to MySQL to speed up execution

```sql
> EXPLAIN select * from customer, nation where c_nationkey = n_nationkey and n_regionkey = 3;
Project(c_custkey="c_custkey", c_name="c_name", c_address="c_address", c_nationkey="c_nationkey", c_phone="c_phone", c_acctbal="c_acctbal", c_mktsegment="c_mktsegment", c_comment="c_comment", n_nationkey="n_nationkey", n_name="n_name", n_regionkey="n_regionkey", n_comment="n_comment")
BKAJoin(condition="c_nationkey = n_nationkey", type="inner")
Gather(concurrent=true)
LogicalView(tables="nation", shardCount=2, sql="SELECT * FROM `nation` AS `nation` WHERE (`n_regionkey` = ?)")
Gather(concurrent=true)
LogicalView(tables="customer_[0-7]", shardCount=8, sql="SELECT * FROM `customer` AS `customer` WHERE (`c_nationkey` IN ('?'))")
```



If BKAJOIN appears in the execution plan, each time BKAJOIN obtains a batch of data from the left table, it will compose an IN query to fetch the associated rows from the right table, and finally execute the JOIN operation. Due to the large amount of data in the left table, it needs to be retrieved many times to complete the query, and the execution is very slow.

The reason why the JOIN cannot be pushed down is that in the current situation, nation is segmented by the primary key n_nationkey, but the JOIN Key of this query is c_custkey, which is different, so the push-down fails.

Considering that the amount of data in the nation (country) table is not large and there are almost no modification operations, it can be rebuilt into the following broadcast table:

```sql
--- Modified ---
CREATE TABLE `nation` (
`n_nationkey` int(11) NOT NULL,
`n_name` varchar(25) NOT NULL,
`n_regionkey` int(11) NOT NULL,
`n_comment` varchar(152) DEFAULT NULL,
PRIMARY KEY (`n_nationkey`)
) BROADCAST; --- Declare as broadcast table
```



After the modification, you can see that JOIN no longer appears in the execution plan, and almost all calculations are pushed down to the storage layer MySQL for execution (in LogicalView), while the upper layer only collects the results and returns them to the user (Gather operator). Performance is greatly enhanced.

```sql
> EXPLAIN select * from customer, nation where c_nationkey = n_nationkey and n_regionkey = 3;
Gather(concurrent=true)
LogicalView(tables="customer_[0-7],nation", shardCount=8, sql="SELECT * FROM `customer` AS `customer` INNER JOIN `nation` AS `nation` ON ((`nation`.`n_regionkey` = ?) AND (`customer`.`c_nationkey` = `nation`.`n_nationkey`))")
```

For more information on the principle and optimization of pushdown, please refer to [Query Rewriting and Pushdown](query-rewriting.md).

increase index
-------------------------

PolarDB-X supports [Global Secondary Index](../../features/topics/gsi.md)

The following is an example of slow SQL to explain how to push down more operators through GSI

```sql
> EXPLAIN select o_orderkey, c_custkey, c_name from orders, customer
where o_custkey = c_custkey and o_orderdate = '2019-11-11' and o_totalprice > 100;
Project(o_orderkey="o_orderkey", c_custkey="c_custkey", c_name="c_name")
HashJoin(condition="o_custkey = c_custkey", type="inner")
Gather(concurrent=true)
LogicalView(tables="customer_[0-7]", shardCount=8, sql="SELECT `c_custkey`, `c_name` FROM `customer` AS `customer`")
Gather(concurrent=true)
LogicalView(tables="orders_[0-7]", shardCount=8, sql="SELECT `o_orderkey`, `o_custkey` FROM `orders` AS `orders` WHERE ((`o_orderdate` = ?) AND (`o_totalprice` > ?))")
```



In the execution plan, orders are split according to o_orderkey and customers are split according to c_custkey. Due to different split dimensions, the JOIN operator cannot be pushed down. Considering that there are a lot of orders with a total price higher than 100 on 2019-11-11, cross-shard JOIN takes a long time, and it is necessary to create a GSI on the orders table so that the JOIN operator can be pushed down. The query uses the four columns o_orderkey, o_custkey, o_orderdate, o_totalprice of the orders table, where o_orderkey, o_custkey are the split keys of the main table and the index table respectively, and o_orderdate, o_totalprice are included in the index as covering columns to avoid returning to the table.

```sql
> create global index i_o_custkey on orders(`o_custkey`) covering(`o_orderdate`, `o_totalprice`)
DBPARTITION BY HASH(`o_custkey`) TBPARTITION BY HASH(`o_custkey`) TBPARTITIONS 4;
```



After adding GSI and forcing the index to be used through force index (i_o_custkey), the cross-shard JOIN becomes a partial JOIN on MySQL (in IndexScan), and the table return operation is avoided by covering columns, and the query performance is improved.

```sql
> EXPLAIN select o_orderkey, c_custkey, c_name from orders force index(i_o_custkey), customer
where o_custkey = c_custkey and o_orderdate = '2019-11-11' and o_totalprice > 100;
Gather(concurrent=true)
IndexScan(tables="i_o_custkey_[0-7],customer_[0-7]", shardCount=8, sql="SELECT `i_o_custkey`.`o_orderkey`, `customer`.`c_custkey`, `customer`.`c_name` FROM `i_o_custkey` AS `i_o_custkey` INNER JOIN `customer` AS `customer` ON (((`i_o_custkey`.`o_orderdate` = ?) AND (`i_o_custkey`.`o_custkey` = `customer`.`c_custkey`)) AND (`i_o_custkey`.`o_totalprice` > ?))")
```

For more details about the use of global secondary indexes, please refer to [Global Secondary Indexes](../../dev-guide/topics/gsi-faq.md).

Execution plan tuning
---------------------------

In most cases, PolarDB-X's query optimizer can automatically generate the best execution plan. However, in a few cases, the generated execution plan may not be good enough due to lack of statistical information, errors, etc. At this time, Hint can be used to intervene in the behavior of the optimizer to generate a better execution plan. The following example will explain the tuning of the execution plan.

```sql
> EXPLAIN select o_orderkey, c_custkey, c_name from orders, customer
where o_custkey = c_custkey and o_orderdate = '2019-11-15' and o_totalprice < 10;
Project(o_orderkey="o_orderkey", c_custkey="c_custkey", c_name="c_name")
HashJoin(condition="o_custkey = c_custkey", type="inner")
Gather(concurrent=true)
LogicalView(tables="customer_[0-7]", shardCount=8, sql="SELECT `c_custkey`, `c_name` FROM `customer` AS `customer`")
Gather(concurrent=true)
LogicalView(tables="orders_[0-7]", shardCount=8, sql="SELECT `o_orderkey`, `o_custkey` FROM `orders` AS `orders` WHERE ((`o_orderdate` = ?) AND (`o_totalprice` < ?))")
```



In fact, on November 15, 2019, the number of orders with a total price of less than 10 yuan was very small, and there were only a few. At this time, using BKAJOIN is a better choice than Hash JOIN (for the introduction of BKAJOIN and Hash JOIN, please refer to [JOIN Optimization and Execution](join-optimizing.md)

Force the optimizer to use BKAJOIN (LookupJOIN) by /\*+TDDL:BKA_JOIN(orders, customer)\*/ Hint as follows:

```sql
> EXPLAIN /*+TDDL:BKA_JOIN(orders, customer)*/ select o_orderkey, c_custkey, c_name from orders, customer
where o_custkey = c_custkey and o_orderdate = '2019-11-15' and o_totalprice < 10;
Project(o_orderkey="o_orderkey", c_custkey="c_custkey", c_name="c_name")
BKAJoin(condition="o_custkey = c_custkey", type="inner")
Gather(concurrent=true)
LogicalView(tables="orders_[0-7]", shardCount=8, sql="SELECT `o_orderkey`, `o_custkey` FROM `orders` AS `orders` WHERE ((`o_orderdate` = ?) AND (`o_totalprice` < ?))")
Gather(concurrent=true)
LogicalView(tables="customer_[0-7]", shardCount=8, sql="SELECT `c_custkey`, `c_name` FROM `customer` AS `customer` WHERE (`c_custkey` IN ('?'))")
```

You can choose to execute the query with the following Hint added:

```sql
/*+TDDL:BKA_JOIN(orders, customer)*/ select o_orderkey, c_custkey, c_name from orders, customer where o_custkey = c_custkey and o_orderdate = '2019-11-15' and o_totalprice < 10;
```

The above operations speed up the SQL query. In order to make Hint work, you can add Hint to the SQL in the application, or a more convenient way is to use the execution plan management (Plan Management) function to fix the execution plan of the SQL. The specific operation is as follows:

```sql
BASELINE FIX SQL /*+TDDL:BKA_JOIN(orders, customer)*/ select o_orderkey, c_custkey, c_name from orders, customer where o_custkey = c_custkey and o_orderdate = '2019-11-15';
```



In this way, for this SQL (parameters can be different), PolarDB-X will adopt the above fixed execution plan. For more information about execution plan management, please refer to [Execution Plan Management](spm.md)

concurrent execution
-------------------------

Users can specify the degree of parallelism through HINT /\*+TDDL:PARALLELISM=4\*/ to make full use of multi-core capabilities to accelerate calculations. Take the following example:

```sql
mysql> explain physical select a.k, count(*) cnt from sbtest1 a, sbtest1 b where a.id = b.k and a.id > 1000 group by k having cnt > 1300 or
der by cnt limit 5, 10;
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| PLAN                                                                                                                                                              |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ExecutorType: AP_LOCAL                                                                                                                                                 |
| The Query's MaxConcurrentParallelism: 2                                                                                                                           |
| Fragment 1                                                                                                                                                        |
|     Shuffle Output layout: [BIGINT, BIGINT] Output layout: [BIGINT, BIGINT]                                                                                       |
|     Output partitioning: SINGLE [] Parallelism: 1                                                                                                                 |
|     TopN(sort="cnt ASC", offset=?2, fetch=?3)                                                                                                                     |
|   Filter(condition="cnt > ?1")                                                                                                                                    |
|     HashAgg(group="k", cnt="COUNT()")                                                                                                                             |
|       BKAJoin(condition="k = id", type="inner")                                                                                                                   |
|         RemoteSource(sourceFragmentIds=[0], type=RecordType(INTEGER_UNSIGNED id, INTEGER_UNSIGNED k))                                                             |
|         Gather(concurrent=true)                                                                                                                                   |
|           LogicalView(tables="[000000-000003].sbtest1_[00-15]", shardCount=16, sql="SELECT `k` FROM `sbtest1` AS `sbtest1` WHERE ((`k` > ?) AND (`k` IN (...)))") |
| Fragment 0                                                                                                                                                        |
|     Shuffle Output layout: [BIGINT, BIGINT] Output layout: [BIGINT, BIGINT]                                                                                       |
|     Output partitioning: SINGLE [] Parallelism: 1 Splits: 16                                                                                                      |
|     LogicalView(tables="[000000-000003].sbtest1_[00-15]", shardCount=16, sql="SELECT `id`, `k` FROM `sbtest1` AS `sbtest1` WHERE (`id` > ?)")                     |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```



The default degree of parallelism is not high. By forcing the specified degree of parallelism, it can be accelerated by using single-machine or multi-machine parallel mode.

```sql
mysql> explain physical /*+TDDL:PARALLELISM=8*/select a.k, count(*) cnt from sbtest1 a, sbtest1 b where a.id = b.k and a.id > 1000 group by k having cnt > 1300 order by cnt limit 5, 10;                                                                                                                                                     |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ExecutorMode: AP_LOCAL                                                                                                                                      |
| Fragment 0 dependency: [] parallelism: 8                                                                                                                    |
| BKAJoin(condition="k = id", type="inner")                                                                                                                   |
|   Gather(concurrent=true)                                                                                                                                   |
|     LogicalView(tables="[000000-000003].sbtest1_[00-15]", shardCount=16, sql="SELECT `id`, `k` FROM `sbtest1` AS `sbtest1` WHERE (`id` > ?)")               |
|   Gather(concurrent=true)                                                                                                                                   |
|     LogicalView(tables="[000000-000003].sbtest1_[00-15]", shardCount=16, sql="SELECT `k` FROM `sbtest1` AS `sbtest1` WHERE ((`k` > ?) AND (`k` IN (...)))") |
| Fragment 1 dependency: [] parallelism: 8                                                                                                                    |
| LocalBuffer                                                                                                                                                 |
|   RemoteSource(sourceFragmentIds=[0], type=RecordType(INTEGER_UNSIGNED id, INTEGER_UNSIGNED k, INTEGER_UNSIGNED k0))                                        |
| Fragment 2 dependency: [0, 1] parallelism: 8                                                                                                                |
| Filter(condition="cnt > ?1")                                                                                                                                |
|   HashAgg(group="k", cnt="COUNT()")                                                                                                                         |
|     RemoteSource(sourceFragmentIds=[1], type=RecordType(INTEGER_UNSIGNED id, INTEGER_UNSIGNED k, INTEGER_UNSIGNED k0))                                      |
| Fragment 3 dependency: [0, 1] parallelism: 1                                                                                                                |
| LocalBuffer                                                                                                                                                 |
|   RemoteSource(sourceFragmentIds=[2], type=RecordType(INTEGER_UNSIGNED k, BIGINT cnt))                                                                      |
| Fragment 4 dependency: [2, 3] parallelism: 1                                                                                                                |
| TopN(sort="cnt ASC", offset=?2, fetch=?3)                                                                                                                   |
|   RemoteSource(sourceFragmentIds=[3], type=RecordType(INTEGER_UNSIGNED k, BIGINT cnt))                                                                      |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------+
```



