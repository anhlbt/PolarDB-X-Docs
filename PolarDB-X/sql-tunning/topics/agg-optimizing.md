Aggregate Optimization and Execution
============================

This article introduces how the optimizer and executor process aggregation (Group-by) to achieve the effect of reducing the amount of data transmission and improving execution efficiency.

basic concept
-------------------------

Aggregate operation (Aggregate, referred to as Agg) semantics is the calculation of aggregating input data according to the specified column of GROUP BY, or the calculation of aggregating all data without grouping. PolarDB-X supports the following aggregation functions:

* COUNT

* SUM

* AVG

* MAX

* MIN

* BIT_OR

* BIT_XOR

* GROUP_CONCAT




Aggregation (Agg)
----------------------------

This article introduces the implementation of Agg without pushdown. If it has been pushed down to LogicalView, the execution method is selected by the storage layer MySQL, and aggregation (Agg) is realized by two main operators HashAgg and SortAgg.

**HashAgg**

HashAgg uses hash tables to achieve aggregation:

1. According to the value of the grouping column of the input row, find the corresponding grouping through Hash.

2. Perform aggregation calculation on the row according to the specified aggregation function.

3. Repeat the above steps until all input rows are processed, and finally output the aggregation result.




```sql
> explain select count(*) from t1 join t2 on t1.id = t2.id group by t1.name,t2.name;
Project(count(*)="count(*)")
HashAgg(group="name,name0", count(*)="COUNT()")
BKAJoin(condition="id = id", type="inner")
Gather(concurrent=true)
LogicalView(tables="t1", shardCount=2, sql="SELECT `id`, `name` FROM `t1` AS `t1`")
Gather(concurrent=true)
LogicalView(tables="t2_[0-3]", shardCount=4, sql="SELECT `id`, `name` FROM `t2` AS `t2` WHERE (`id` IN ('?'))")
```

In the Explain result, the HashAgg operator also contains the following key information:

* group: Indicates the GROUP BY field. In the example, name and name0 respectively refer to the name columns of the t1 and t2 tables. When the same alias exists, it will be distinguished by the suffix number.

* Aggregation function: The equal sign (=) is the output column name corresponding to the aggregation function, followed by the corresponding calculation method. In the example count(\*)="COUNT()" , the first count(\*) corresponds to the output column name, and the subsequent COUNT() means to count its input data.


HashAgg correspondence can be turned off by Hint: `/*+TDDL:cmd_extra(ENABLE_HASH_AGG=false)*/`

**SortAgg**

SortAgg completes the aggregation for each group in turn when the input data is sorted by the grouping column.

* Guarantees that the input is sorted by the specified grouping column (e.g. might see MergeSort or MemSort).

* Read in the input data line by line, if the grouping is the same as the current grouping, perform aggregation calculation on it.

* If the grouping is different from the current grouping, output the aggregation result on the current grouping.


Compared with HashAgg, SortAgg only needs to process one group at a time, and the memory consumption is very small; in contrast, HashAgg needs to store all groups in memory, which consumes more memory.

```sql
> explain select count(*) from t1 join t2 on t1.id = t2.id group by t1.name,t2.name order by t1.name, t2.name;
Project(count(*)="count(*)")
MemSort(sort="name ASC,name0 ASC")
HashAgg(group="name,name0", count(*)="COUNT()")
BKAJoin(condition="id = id", type="inner")
Gather(concurrent=true)
LogicalView(tables="t1", shardCount=2, sql="SELECT `id`, `name` FROM `t1` AS `t1`")
Gather(concurrent=true)
LogicalView(tables="t2_[0-3]", shardCount=4, sql="SELECT `id`, `name` FROM `t2` AS `t2` WHERE (`id` IN ('?'))")
```

SortAgg correspondence can be turned off by Hint: `/*+TDDL:cmd_extra(ENABLE_SORT_AGG=false)*/`

**Two-stage aggregation optimization**

Two-stage aggregation, that is, by splitting Agg into two stages of partial aggregation (Partial Agg) and final aggregation (Final Agg), first aggregate partial result sets, and then aggregate these partial aggregation results to obtain the overall aggregation result .

In the SQL of the following example, the partial aggregation (PartialAgg) split from HashAgg will be pushed down to each sub-table on MySQL, and the AVG function in it will also be split into SUM and COUNT to realize two-stage calculation:

```sql
> explain select avg(age) from t2 group by name
Project(avg(age)="sum_pushed_sum / sum_pushed_count")
HashAgg(group="name", sum_pushed_sum="SUM(pushed_sum)", sum_pushed_count="SUM(pushed_count)")
Gather(concurrent=true)
LogicalView(tables="t2_[0-3]", shardCount=4, sql="SELECT `name`, SUM(`age`) AS `pushed_sum`, COUNT(`age`) AS `pushed_count` FROM `t2` AS `t2` GROUP BY `name`")
```

The optimization of two-stage aggregation can greatly reduce the amount of data transmission and improve execution efficiency.

In general, HashAgg tends to be selected for aggregation in most scenarios, and SortAgg is suitable for aggregation only in the following scenarios:

1. There is a lot of data, and the memory is seriously insufficient.

2. The input of the aggregation operator has been sorted according to the Group By column, so SortAgg does not need additional sorting, and the execution efficiency will be higher.

3. When the data is seriously skewed, resulting in low HashAgg execution efficiency, SortAgg is preferred




