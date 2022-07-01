统计信息查询 
===========================

本文介绍了用于查询实时统计信息的语句。

* SHOW [FULL] STATS

* SHOW DB STATUS

* SHOW TABLE STATUS

* SHOW TABLE INFO [name]




SHOW [FULL] STATS 
----------------------------------------

查看整体的统计信息，这些信息都是瞬时值。 注意不同版本的PolarDB-X`SHOW FULL STATS`的结果是有区别的。

示例：

```sql
mysql> show stats;
+------+---------+----------+-------------------+------------------+------------------------+--------------------+--------+------------+--------------+---------------+----------------+---------------+---------------+--------------+
| QPS  | RDS_QPS | SLOW_QPS | PHYSICAL_SLOW_QPS | ERROR_PER_SECOND | MERGE_QUERY_PER_SECOND | ACTIVE_CONNECTIONS | RT(MS) | RDS_RT(MS) | NET_IN(KB/S) | NET_OUT(KB/S) | THREAD_RUNNING | DDL_JOB_COUNT | BACKFILL_ROWS | CHECKED_ROWS |
+------+---------+----------+-------------------+------------------+------------------------+--------------------+--------+------------+--------------+---------------+----------------+---------------+---------------+--------------+
| 0.00 |    0.00 |     0.00 |              0.00 |             0.00 |                   0.00 |                  1 |   0.00 |       0.00 |         0.00 |          0.00 |              1 |             0 |             0 |            0 |
+------+---------+----------+-------------------+------------------+------------------------+--------------------+--------+------------+--------------+---------------+----------------+---------------+---------------+--------------+
mysql> show full stats;
+------+---------+----------+-------------------+------------------+----------------------+------------------------+--------------------+------------------------------+--------+------------+--------------+---------------+----------------+----------------------+-----------------+----------------------------+-----------------------+------------------------------+-------------------------+--------------------------+---------------------+-------+---------+-------------+------------+
| QPS  | RDS_QPS | SLOW_QPS | PHYSICAL_SLOW_QPS | ERROR_PER_SECOND | VIOLATION_PER_SECOND | MERGE_QUERY_PER_SECOND | ACTIVE_CONNECTIONS | CONNECTION_CREATE_PER_SECOND | RT(MS) | RDS_RT(MS) | NET_IN(KB/S) | NET_OUT(KB/S) | THREAD_RUNNING | HINT_USED_PER_SECOND | HINT_USED_COUNT | AGGREGATE_QUERY_PER_SECOND | AGGREGATE_QUERY_COUNT | TEMP_TABLE_CREATE_PER_SECOND | TEMP_TABLE_CREATE_COUNT | MULTI_DB_JOIN_PER_SECOND | MULTI_DB_JOIN_COUNT | CPU   | FREEMEM | FULLGCCOUNT | FULLGCTIME |
+------+---------+----------+-------------------+------------------+----------------------+------------------------+--------------------+------------------------------+--------+------------+--------------+---------------+----------------+----------------------+-----------------+----------------------------+-----------------------+------------------------------+-------------------------+--------------------------+---------------------+-------+---------+-------------+------------+
| 1.63 |    1.68 |     0.03 |              0.03 |             0.02 |                 0.00 |                   0.00 |                  6 |                         0.01 | 157.13 |      51.14 |       134.33 |          1.21 |              1 |                 0.00 |              54 |                       0.00 |                   663 |                         0.00 |                     512 |                     0.00 |                 516 | 0.09% |   6.96% |       76446 |   21326906 |
+------+---------+----------+-------------------+------------------+----------------------+------------------------+--------------------+------------------------------+--------+------------+--------------+---------------+----------------+----------------------+-----------------+----------------------------+-----------------------+------------------------------+-------------------------+--------------------------+---------------------+-------+---------+-------------+------------+
1 row in set (0.01 sec)
```



重要列说明：

* **QPS** ：应用到PolarDB-X的QPS，通常称为逻辑QPS；

* **RDS_QPS** ：PolarDB-X到RDS的QPS，通常称为物理QPS；

* **ERROR_PER_SECOND** ：每秒的错误数，包含SQL语法错误，主键冲突，系统错误，连通性错误等各类错误总和；

* **VIOLATION_PER_SECOND** ：每秒的主键或者唯一键冲突；

* **MERGE_QUERY_PER_SECCOND** ：通过分库分表，从多表中进行的查询；

* **ACTIVE_CONNECTIONS** ：正在使用的连接；

* **CONNECTION_CREATE_PER_SECCOND** ：每秒创建的连接数；

* **RT(MS)** ：应用到PolarDB-X的响应时间，通常称为逻辑RT（响应时间）；

* **RDS_RT(MS)** ：PolarDB-X到RDS/MySQL的响应时间，通常称为物理RT；

* **NET_IN(KB/S)** ：PolarDB-X收到的网络流量；




<!-- -->

* **NET_OUT(KB/S)** ：PolarDB-X输出的网络流量；

* **THREAD_RUNNING** ：正在运行的线程数；

* **HINT_USED_PER_SECOND** ：每秒带HINT的查询的数量；

* **HINT_USED_COUNT** ：启动到现在带HINT的查询总量；

* **AGGREGATE_QUERY_PER_SECCOND** ：每秒聚合查询的频次；

* **AGGREGATE_QUERY_COUNT** ：聚合查询总数（历史累计数据）；

* **TEMP_TABLE_CREATE_PER_SECCOND** ：每秒创建的临时表的数量；

* **TEMP_TABLE_CREATE_COUNT** ：启动到现在创建的临时表总数量；

* **MULTI_DB_JOIN_PER_SECCOND** ：每秒跨库JOIN的数量；

* **MULTI_DB_JOIN_COUNT** ：启动到现在跨库JOIN的总量。




SHOW DB STATUS 
-----------------------------------

用于查看物理库容量/性能信息，所有返回值为实时信息。 容量信息通过MySQL系统表获得，与真实容量情况可能有差异。

示例：

```sql
mysql> show db status;
+------+---------------------------+--------------------+-------------------+------------+--------+----------------+
| ID   | NAME                      | CONNECTION_STRING  | PHYSICAL_DB       | SIZE_IN_MB | RATIO  | THREAD_RUNNING |
+------+---------------------------+--------------------+-------------------+------------+--------+----------------+
|    1 | drds_db_1516187088365daui | 100.100.64.1:59077 | TOTAL             |  13.109375 | 100%   | 3              |
|    2 | drds_db_1516187088365daui | 100.100.64.1:59077 | drds_db_xzip_0000 |   1.578125 | 12.04% |                |
|    3 | drds_db_1516187088365daui | 100.100.64.1:59077 | drds_db_xzip_0001 |     1.4375 | 10.97% |                |
|    4 | drds_db_1516187088365daui | 100.100.64.1:59077 | drds_db_xzip_0002 |     1.4375 | 10.97% |                |
|    5 | drds_db_1516187088365daui | 100.100.64.1:59077 | drds_db_xzip_0003 |     1.4375 | 10.97% |                |
|    6 | drds_db_1516187088365daui | 100.100.64.1:59077 | drds_db_xzip_0004 |   1.734375 | 13.23% |                |
|    7 | drds_db_1516187088365daui | 100.100.64.1:59077 | drds_db_xzip_0005 |   1.734375 | 13.23% |                |
|    8 | drds_db_1516187088365daui | 100.100.64.1:59077 | drds_db_xzip_0006 |   2.015625 | 15.38% |                |
|    9 | drds_db_1516187088365daui | 100.100.64.1:59077 | drds_db_xzip_0007 |   1.734375 | 13.23% |                |
+------+---------------------------+--------------------+-------------------+------------+--------+----------------+
```



重要列说明：

* **NAME** ：代表一个PolarDB-XDB，此处显示的是PolarDB-X内部标记，与PolarDB-XDB名称不同；

* **CONNECTION_STRING** ：分库的连接信息；

* **PHYSICAL_DB** ：分库名称，`TOTAL`行代表一个PolarDB-XDB中所有分库容量的总和；

* **SIZE_IN_MB** ：分库中数据占用的空间，单位为MB；

* **RATIO** ：单个分库数据量在当前PolarDB-XDB总数据量中的占比；

* **THREAD_RUNNING** ：物理数据库实例当前正在执行的线程情况，含义与MySQL语句`SHOW GLOBAL STATUS`返回值的含义相同，详情请参考 [MySQL 文档](https://dev.mysql.com/doc/refman/5.7/en/server-status-variables.html) 。




SHOW TABLE STATUS 
--------------------------------------

获取表的信息，该指令聚合了底层各个物理分表的数据。

示例：

```sql
mysql> SHOW TABLE STATUS;
+---------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+-------------+------------+--------------------+----------+----------------+---------+
| NAME    | ENGINE | VERSION | ROW_FORMAT | ROWS | AVG_ROW_LENGTH | DATA_LENGTH | MAX_DATA_LENGTH | INDEX_LENGTH | DATA_FREE | AUTO_INCREMENT | CREATE_TIME         | UPDATE_TIME | CHECK_TIME | COLLATION          | CHECKSUM | CREATE_OPTIONS | COMMENT |
+---------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+-------------+------------+--------------------+----------+----------------+---------+
| sbtest1 | InnoDB |      10 | Dynamic    |    0 |              0 |     1310720 |               0 |            0 |         0 |              0 | 2021-07-20 15:39:37 | NULL        | NULL       | utf8mb4_general_ci | NULL     |                |         |
| t1      | InnoDB |      10 | Dynamic    |    0 |              0 |     2621440 |               0 |      2621440 |         0 |         200000 | 2021-07-26 20:11:15 | NULL        | NULL       | utf8mb4_general_ci | NULL     |                |         |
+---------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+-------------+------------+--------------------+----------+----------------+---------+
```



重要列详解：

* **NAME** ：表名称；

* **ENGINE** ：表的存储引擎；

* **VERSION** ：表的存储引擎的版本；

* **ROW_FORMAT** ：行格式，主要是Dynamic、Fixed、Compressed这三种格式。动态（Dynamic）行的行长度可变，例如VARCHAR或BLOB类型字段；固定（Fixed）行是指行长度不变，例如CHAR和INTEGER类型字段；

* **ROWS** ：表中的行数；

* **AVG_ROW_LENGTH** ：平均每行包括的字节数；

* **DATA_LENGTH** ：整个表的数据量（单位：字节）；

* **MAX_DATA_LENGTH** ：表可以容纳的最大数据量；

* **INDEX_LENGTH** ：索引占用磁盘的空间大小 ；




<!-- -->

* **CREATE_TIME** ：表的创建时间；

* **UPDATE_TIME** ：表的最近更新时间；

* **COLLATION** ：表的默认字符集和字符排序规则；

* **CREATE_OPTIONS** ：指表创建时的其他所有选项。




SHOW TABLE INFO [name]
---------------------------------------------

获取各个分表的数据量信息。

示例：

```sql
mysql> show table info  sbtest1;
+----+--------------+-----------------+------------+
| ID | GROUP_NAME   | TABLE_NAME      | SIZE_IN_MB |
+----+--------------+-----------------+------------+
|  0 | test1_000000 | sbtest1_wo5k_00 | 0.01562500 |
|  1 | test1_000000 | sbtest1_wo5k_01 | 0.01562500 |
|  2 | test1_000005 | sbtest1_wo5k_10 | 0.01562500 |
|  3 | test1_000005 | sbtest1_wo5k_11 | 0.01562500 |
|  4 | test1_000010 | sbtest1_wo5k_20 | 0.01562500 |
|  5 | test1_000010 | sbtest1_wo5k_21 | 0.01562500 |
|  6 | test1_000015 | sbtest1_wo5k_30 | 0.01562500 |
|  7 | test1_000015 | sbtest1_wo5k_31 | 0.01562500 |
|  8 | test1_000020 | sbtest1_wo5k_40 | 0.01562500 |
|  9 | test1_000020 | sbtest1_wo5k_41 | 0.01562500 |
| 10 | test1_000025 | sbtest1_wo5k_50 | 0.01562500 |
| 11 | test1_000025 | sbtest1_wo5k_51 | 0.01562500 |
| 12 | test1_000030 | sbtest1_wo5k_60 | 0.01562500 |
| 13 | test1_000030 | sbtest1_wo5k_61 | 0.01562500 |
| 14 | test1_000035 | sbtest1_wo5k_70 | 0.01562500 |
| 15 | test1_000035 | sbtest1_wo5k_71 | 0.01562500 |
+----+--------------+-----------------+------------+
```



重要列详解：

* **ID** ：标识；

* **GROUP_NAME** ：分库名；

* **TABLE_NAME** ：物理分表名；

* **SIZE_IN_MB** ：表大小；


