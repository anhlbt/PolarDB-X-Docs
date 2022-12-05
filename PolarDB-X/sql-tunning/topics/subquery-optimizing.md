Subquery optimization and execution
=============================

A subquery is a query that nests another SELECT statement in the WHERE clause or HAVING clause of the parent query. This article mainly introduces how to subquery.

basic concept
-------------------------

According to whether there are associated items, subqueries can be divided into non-correlated subqueries and correlated subqueries. A non-correlated subquery means that the execution of the subquery does not depend on the variables of the outer query. This kind of subquery generally only needs to be calculated once; while there are variables referenced from the outer query in the correlated subquery. Logically, this kind of subquery needs to be calculated every time. Bring in the corresponding variables and calculate multiple times.

```sql
/* Example: non-correlated subquery */
SELECT * FROM lineitem WHERE l_partkey IN (SELECT p_partkey FROM part);
/* Example: associated subquery (l_suppkey is an associated item) */
SELECT * FROM lineitem WHERE l_partkey IN (SELECT ps_partkey FROM partsupp WHERE ps_suppkey = l_suppkey);
```

PolarDB-X subqueries support most of the subquery writing methods, see [SQL Usage Limitation](../../dev-guide/topics/limitation.md) for details.

subquery execution
--------------------------

For most common subquery forms, PolarDB-X can rewrite them into efficient SemiJoin or similar JOIN-based calculation methods. The benefits of doing this are obvious. When the amount of data is large, there is no need to actually bring in different parameters for loop iterations, which greatly reduces the execution cost. This query rewriting technique is called subquery de-association (Unnesting).

In the following example, the two subqueries are de-associated, and you can see that JOIN is used instead of subqueries in the execution plan.

```sql
> EXPLAIN SELECT p_partkey, (
SELECT COUNT(ps_partkey) FROM partsupp WHERE ps_suppkey = p_partkey
) supplier_count FROM part;
Project(p_partkey="p_partkey", supplier_count="CASE(IS NULL($10), 0, $9)", cor=[$cor0])
HashJoin(condition="p_partkey = ps_suppkey", type="left")
Gather(concurrent=true)
LogicalView(tables="part_[0-7]", shardCount=8, sql="SELECT * FROM `part` AS `part`")
Project(count(ps_partkey)="count(ps_partkey)", ps_suppkey="ps_suppkey", count(ps_partkey)2="count(ps_partkey)")
HashAgg(group="ps_suppkey", count(ps_partkey)="SUM(count(ps_partkey))")
Gather(concurrent=true)
LogicalView(tables="partsupp_[0-7]", shardCount=8, sql="SELECT `ps_suppkey`, COUNT(`ps_partkey`) AS `count(ps_partkey)` FROM `partsupp` AS `partsupp` GROUP BY `ps_suppkey`")
```



```sql
> EXPLAIN SELECT p_partkey, (
SELECT COUNT(ps_partkey) FROM partsupp WHERE ps_suppkey = p_partkey
) supplier_count FROM part;
Project(p_partkey="p_partkey", supplier_count="CASE(IS NULL($10), 0, $9)", cor=[$cor0])
HashJoin(condition="p_partkey = ps_suppkey", type="left")
Gather(concurrent=true)
LogicalView(tables="part_[0-7]", shardCount=8, sql="SELECT * FROM `part` AS `part`")
Project(count(ps_partkey)="count(ps_partkey)", ps_suppkey="ps_suppkey", count(ps_partkey)2="count(ps_partkey)")
HashAgg(group="ps_suppkey", count(ps_partkey)="SUM(count(ps_partkey))")
Gather(concurrent=true)
LogicalView(tables="partsupp_[0-7]", shardCount=8, sql="SELECT `ps_suppkey`, COUNT(`ps_partkey`) AS `count(ps_partkey)` FROM `partsupp` AS `partsupp` GROUP BY `ps_suppkey`")
```



In some scenarios, PolarDB-X cannot disassociate subqueries, and iterative execution will be used in this case. If the amount of data in the outer query is large, iterative execution may be very slow.

In the following example, due to the existence of OR l_partkey < 50, the subquery cannot be disassociated, so iterative execution is used:

```sql
> EXPLAIN SELECT * FROM lineitem WHERE l_partkey IN (SELECT ps_partkey FROM partsupp WHERE ps_suppkey = l_suppkey) OR l_partkey IS NOT
Filter(condition="IS(in,[$1])[29612489] OR l_partkey < ?0")
Gather(concurrent=true)
LogicalView(tables="QIMU_0000_GROUP,QIMU_0001_GROUP.lineitem_[0-7]", shardCount=8, sql="SELECT * FROM `lineitem` AS `lineitem`")
>> individual correlate subquery : 29612489
Gather(concurrent=true)
LogicalView(tables="QIMU_0000_GROUP,QIMU_0001_GROUP.partsupp_[0-7]", shardCount=8, sql="SELECT * FROM (SELECT `ps_partkey` FROM `partsupp` AS `partsupp` WHERE (`ps_suppkey` = `l_suppkey`)) AS `t0` WHERE (((`l_partkey` = `ps_partkey`) OR (`l_partkey` IS NULL)) OR (`ps_partkey` IS NULL))")
```

In this case, it is recommended to rewrite the SQL to remove the OR condition of the subquery.

