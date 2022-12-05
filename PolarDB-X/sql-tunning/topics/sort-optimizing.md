Sort Optimization and Execution
============================

This article introduces how to order (Order-by) operators to achieve the effect of reducing the amount of data transmission and improving execution efficiency.

basic concept
-------------------------

The sort operation (Sort) semantics is to sort the input according to the specified ORDER BY column. This article introduces the implementation of Sort operators that do not push down. If it has been pushed down to LogicalView, the execution method is selected by the storage layer MySQL.

Sort
-----------------------------

The sorting operators in PolarDB-X mainly include MemSort, TopN, and MergeSort.

**MemSort**

The general sorting in PolarDB-X is implemented as the MemSort operator, which runs the Quick Sort algorithm in memory. The following is an example using the MemSort operator:

```sql
> explain select t1.name from t1 join t2 on t1.id = t2.id order by t1.name,t2.name;
Project(name="name")
MemSort(sort="name ASC,name0 ASC")
Project(name="name", name0="name0")
BKAJoin(condition="id = id", type="inner")
Gather(concurrent=true)
LogicalView(tables="t1", shardCount=2, sql="SELECT `id`, `name` FROM `t1` AS `t1`")
Gather(concurrent=true)
LogicalView(tables="t2_[0-3]", shardCount=4, sql="SELECT `id`, `name` FROM `t2` AS `t2` WHERE (`id` IN ('?'))")
```



**TopN**

When ORDER BY and LIMIT appear together in SQL, the Sort operator and Limit operator will be combined into a TopN operator.

The TopN operator maintains a maximum or minimum heap. According to the value of the sort key, the largest or smallest N rows of data are always kept in the heap. When all input data has been processed, N rows (or less than N) left in the heap are the desired result.

```sql
> explain select t1.name from t1 join t2 on t1.id = t2.id order by t1.name,t2.name limit 10;
Project(name="name")
TopN(sort="name ASC,name0 ASC", offset=0, fetch=?0)
Project(name="name", name0="name0")
BKAJoin(condition="id = id", type="inner")
Gather(concurrent=true)
LogicalView(tables="t1", shardCount=2, sql="SELECT `id`, `name` FROM `t1` AS `t1`")
Gather(concurrent=true)
LogicalView(tables="t2_[0-3]", shardCount=4, sql="SELECT `id`, `name` FROM `t2` AS `t2` WHERE (`id` IN ('?'))")
```



**MergeSort**

Usually, as long as the semantics allow, the sorting operation in SQL will be pushed down to MySQL for execution, while the PolarDB-X execution layer only performs the final merge operation, that is, MergeSort. Strictly speaking, MergeSort is not just sorting, but also a data redistribution operator (similar to Gather). The following SQL is to sort the t1 table. After the optimization of the PolarDB-X query optimizer, the Sort operator is pushed down to each MySQL shard for execution, and finally only the merge operation is performed on the upper layer.

```sql
> explain select name from t1 order by name;
MergeSort(sort="name ASC")
LogicalView(tables="t1", shardCount=2, sql="SELECT `name` FROM `t1` AS `t1` ORDER BY `name`")
```

Compared with MemSort, the MergeSort algorithm can reduce the memory consumption of the PolarDB-X layer and fully utilize the computing power of the MySQL layer.

