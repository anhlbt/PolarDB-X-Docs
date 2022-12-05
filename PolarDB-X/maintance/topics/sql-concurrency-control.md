SQL throttling
==========================

In order to deal with problems such as sudden database request traffic, statement access with high resource consumption, and changes in the SQL access model, PolarDB-X provides a node-level SQL current limiting function to limit the execution of SQL that causes the above problems, thereby ensuring instance security. Continuous and stable operation. This article describes how to use the SQL current limiting function.

Create a throttling rule
---------------------------

* **grammar**

```sql
CREATE CCL_RULE [ IF NOT EXISTS ] `ccl_rule_name`
ON `database`.`table`
TO '<usename>'@'<host>'
FOR { UPDATE | SELECT | INSERT | DELETE }
[ filter_options ]
with_options
filter_options:
[ FILTER  BY KEYWORD('KEYWORD1', 'KEYWORD2',...) ]
[ FILTER  BY TEMPLATE('template_id') ]
with_options:
WITH MAX_CONCURRENCY = value1 [ , WAIT_QUEUE_SIZE = value2 ] [ , WAIT_TIMEOUT = value3 ] [ ,FAST_MATCH = { 0 , 1 }]
```



| Parameter || Mandatory | Description |
|------------|-------------------------------------|------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Current limiting rule matching parameters | ```ccl_rule_name``` | Mandatory | The name of the current limiting rule. **Note** In order to avoid conflicts between names and SQL keywords, it is recommended to add a backtick (\`) before and after the rule name |
| Matching parameters of the current limiting rule | ```database`.`table``` | Mandatory | The name of the database and data table, an asterisk (\*) is supported to indicate any match. **Note** To avoid name conflicts with SQL keywords, it is recommended to add a backtick (\`) before and after the database table name. |
| Matching parameters of current limiting rules | `'<usename>'@'<host>'` | Required | Account name. The Host part supports the use of a percent sign (%) to indicate any match. |
| Matching parameters of current limiting rules | `UPDATE | SELECT | INSERT | DELETE` | Mandatory | SQL statement type. Currently UPDATE, SELECT, INSERT and DELETE types are supported. **Note** Each rate limiting rule supports only one type of SQL statement. |
| Matching parameters of current limiting rules | `[ filter_options ]` | Optional | Filtering conditions, including the following two conditions are supported: * Keyword (KEYWORD): When viewing the current limiting rules, the keyword list will be converted into The string format of `["kwd1","kw2","kw3"...]` supports up to 512 characters. **Description** * If the keyword is a parameter value in an SQL statement, the match is case-sensitive. * If the keyword is another word in the SQL statement, the matching is case-insensitive. * Template (TEMPLATE): The template number is the `sql_code` value in the SQL log, which is the hash value of the parameterized SQL statement (SQL template) expressed in hexadecimal. You can view the template ID with the SHOW FULL PROCESSLIST and EXPLAIN commands. |
| Current-limiting rule behavior control parameters | `with_options` | Mandatory | The WITH option supports the following 4 parameters to control the behavior of the current-limiting rule: * MAX_CONCURRENCY: the maximum concurrency of SQL statements matching the current-limiting rule, after exceeding Enter the waiting queue. Value range: \[0\~2^31^ - 1\], the default value is 0. * WAIT_QUEUE_SIZE: The maximum waiting queue length after exceeding the concurrency. When the waiting queue length exceeds this value, the SQL statement will report an error. Statements in the queue still occupy thread resources, and may also cause memory exhaustion when there are too many queues. Value range: \[0\~2^31^ - 1\], the default value is 0. * WAIT_TIMEOUT: The maximum waiting time of the SQL statement in the waiting queue. After the waiting time is exceeded, the SQL statement will report an error. Value range: \[0\~2^31^ - 1\], the unit is second, and the default value is 600. * FAST_MATCH: Whether to enable Cache to speed up matching. After it is enabled, PolarDB-X 2.0 will use the template number as part of the Cache key, and the matching result as the value to cache to speed up the matching speed. Value range: 0 means off, 1 means on, and the default is on. **Description** * When creating a traffic limiting rule, at least one of the above four behavior control parameters needs to be selected and passed in. * When MAX_CONCURRENCY is the default value (0), it may cause all matched SQL to return an error. At this point, it is recommended that you explicitly specify a value other than 0 for this parameter. * PolarDB-X 2.0 is a distributed cloud-native database. The computing layer consists of multiple nodes, so the sum of the concurrency of each node is the maximum concurrency of the entire instance. In the case of unbalanced load, the limited SQL concurrency of the entire instance may not reach the maximum concurrency. |
[Parameter Description]


**Description** Only when an SQL statement satisfies all the matching parameter conditions, the traffic will be limited according to the WITH option of the rule.


* **Current limit result**

After a SQL matches the rule, according to the parameters configured in the WITH option in the current limiting rule, the following results will appear:
* **RUN (runnable)**

If the concurrency degree has not reached the maximum concurrency degree (that is, the MAX_CONCURRENCY parameter value), the normal execution of the SQL will not be restricted.


* **WAIT (waiting)**

If the concurrency has reached the maximum concurrency, but the waiting queue length has not reached the maximum length (that is, the WAIT_QUEUE_SIZE parameter value), the SQL enters the waiting state until it enters the runnable (RUN) state, or waits for a timeout (WAIT_TIMEOUT) state.

You can run the following command to view the waiting SQL statements that match the current limiting rule:

```sql
mysql> SHOW FULL PROCESSLIST;
```



An example of the returned result is as follows:

```sql
+----+---------------+-----------------+----------+-------------------------------+------+-------+-----------------------+-----------------+
| ID | USER          | HOST            | DB       | COMMAND                       | TIME | STATE | INFO                  | SQL_TEMPLATE_ID |
+----+---------------+-----------------+----------+-------------------------------+------+-------+-----------------------+-----------------+
|  2 | polardbx_root | ***.*.*.*:62787 | polardbx | Query                         |    0 |       | show full processlist | NULL            |
|  1 | polardbx_root | ***.*.*.*:62775 | polardbx | Query(Waiting-selectrulereal) |   12 |       | select 1              | 9037e5e2        |
+----+---------------+-----------------+----------+-------------------------------+------+-------+-----------------------+-----------------+
2 rows in set (0.08 sec)
```



From the above query results, it can be seen that the SQL statement `select 1` is in the waiting (Waiting) state due to the current limiting rule `selectrulereal`.


* **WAIT_TIMEOUT (wait for timeout)**

After the SQL statement enters the waiting state, if the waiting time exceeds the maximum waiting time (that is, the value of the WAIT_TIMEOUT parameter), the statement will return an error.

For example, if a current limiting rule with a maximum waiting time of 10 seconds is set, an error will be reported when the `SELECT sleep(11)` statement is executed due to waiting timeout. The example is as follows:

```sql
ERROR 3009 (HY000): [11a07e23fd800000][30.225.180.55:8527][polardbx]Exceeding the max concurrency 0 of ccl rule selectrulereal after waiting for 10060 ms
```



* **KILL (end)**

Both the concurrency degree and the waiting queue length have reached the maximum value, and the client will receive an error message indicating that the maximum concurrency degree has been exceeded. The error message will include the name of the matching current limiting rule.

For example, when the `SELECT 1;` command is executed after the concurrency and waiting queue length have reached the maximum value, the following error will appear:

```sql
ERROR 3009 (HY000): [11a07c4425c00000][**.***.***.**:8527][polardbx]Exceeding the max concurrency 0 of ccl rule selectrulereal
```



The above results indicate that the execution of the SQL statement `SELECT 1;` failed because it exceeded the maximum concurrency set by the current limiting rule `selectrulereal`.







* **Example**

Suppose you need to create a rule named `selectrule`, which is used to restrict the SQL statement initiated by the `'ccltest'@'%'` user, which contains the `cclmatched` keyword, and executes the SELECT operation on any table. The maximum concurrency is set to 10.

The rule creation statement is as follows:

```sql
CREATE CCL_RULE IF NOT EXISTS `selectrule` ON *.* TO 'ccltest'@'%'
FOR SELECT
FILTER BY KEYWORD('cclmatched')
WITH MAX_CONCURRENCY=10;
```






View current limiting rules
---------------------------

* **grammar**
* **View the specified current limiting rules**

The syntax is as follows:

```sql
SHOW CCL_RULE `ccl_rule_name1` [, `ccl_rule_name2` ]
```



* **View all throttling rules**

The syntax is as follows:

```sql
SHOW CCL_RULES
```






* **Example**

Use the following command to view all current limiting rules under the current database:

```sql
mysql> SHOW CCL_RULES \G
```



The returned results are as follows:

```sql
*************************** 1. row ***************************
NO.: 1
RULE_NAME: selectrulereal
RUNNING: 2
WAITING: 29
KILLED: 0
MATCH_HIT_CACHE: 21374
TOTAL_MATCH: 21406
ACTIVE_NODE_COUNT: 2
MAX_CONCURRENCY_PER_NODE: 1
WAIT_QUEUE_SIZE_PER_NODE: 100
WAIT_TIMEOUT: 600
FAST_MATCH: 1
SQL_TYPE: SELECT
USER: ccltest@%
TABLE: *.*
KEYWORDS: ["SELECT"]
TEMPLATEID: NULL
CREATED_TIME: 2020-11-26 17:04:08
```



| parameter | description |
|--------------------------|-----------------------------------------------------------|
| NO. | Matching priority, the smaller the number, the higher the priority. |
| RULE_NAME | The name of the current limiting rule. |
| RUNNING | The number of SQL statements that match the current limiting rule and are executed normally. |
| WAITING | The number of queries that match the throttling rule and are waiting in the queue. |
| KILLED | The number of SQL statements that match the current limiting rule and are KILLed. |
| MATCH_HIT_CACHE | The number of SQL statements that match the current limiting rule and hit the cache. |
| TOTAL_MATCH | The total number of matches to this rate limiting rule. |
| ACTIVE_NODE_COUNT | The number of nodes in the compute tier with SQL throttling enabled. |
| MAX_CONCURRENCY_PER_NODE | Concurrency per compute node. |
| WAIT_QUEUE_SIZE_PER_NODE | The maximum length of the wait queue on each compute node. |
| WAIT_TIMEOUT | The maximum waiting time for SQL statements in the waiting queue. |
| FAST_MATCH | Whether to enable caching to speed up matching. |
| SQL_TYPE | SQL statement type. |
| USER | username. |
| TABLE | A database table. |
| KEYWORDS | List of keywords. |
| TEMPLATEID | The ID of the SQL template. |
| CREATED_TIME | Creation time (local time) in `yyyy-MM-dd HH:mm:ss` format. |
[Parameter Description]








Delete the throttling rule
---------------------------

**Note** The deleted current limiting rule will be invalid immediately, and all the SQL statements in the waiting queue under this rule will be executed normally.

* Delete the specified current limiting rule:

```sql
DROP CCL_RULE [ IF EXISTS ] `ccl_rule_name1` [, `ccl_rule_name2`, ...]
```



* Delete all throttling rules:

```sql
CLEAR CCL_RULES
```






Slow SQL throttling
---------------------------

* One-click opening is classified according to the statement type, the default is the SELECT type, and the commands of the same statement type have an update function. The grammatical structure is as follows:

```sql
SLOW_SQL_CCL GO [ SQL_TYPE [MAX_CONCURRENCY] [SLOW_SQL_TIME] [MAX_CCL_RULE]]
```


* SQL_TYPE value: ALL, SELECT, UPDATE, INSERT, the default is SELECT.

* The default value of MAX_CONCURRENCY is half of the number of CPU cores.

* The default value of SLOW_SQL_TIME is the value of the system parameter SLOW_SQL_TIME.

* The default value of MAX_CCL_RULE is 1000.


action:
<!-- -->

* Traverse the session of the entire instance, and identify the TemlateId of the slow SQL of the statement type.

* Create a current-limiting trigger for slow SQL, named: _SYSTEM_SLOW_SQL_CCL_TRIGGER_{SQL_TYPE}_.

* Pass the TemplateId of the slow SQL to the current limiting trigger, and the current limiting trigger will create a current limiting rule.

* Kill all slow TemplateId queries of this statement type.




* One-click shutdown deletes the current limit trigger created by SLOW_SQL_CCL, and the current limit rule created by the current limit trigger will also be deleted. The grammatical structure is as follows:

```sql
SLOW_SQL_CCL BACK
```



* Check current limit situation. The grammatical structure is as follows:

```sql
SLOW_SQL_CCL SHOW
```

An inner join with the template ID as the join key in plan_cache and ccl_rules.

* ![Check current limit situation](../images/p333269.png)

* How to intervene the threshold of slow SQL?
* Set syntax in SLOW_SQL_CCL GO.

* Set the user variable slow_sql_time before enabling SQL current limiting with one click. as follows:

```sql
set @slow_sql_time=2000;
slow_sql_ccl go;
```




**Description** The following setting methods will be overwritten by the previous setting methods.




