SET statement variable setting
==============================

You can use the SET statement to set various variables, including user-defined variables, session variables, and global variables.

grammar
-----------------------

```sql
SET variable = expr [, variable = expr] ...
variable: {
user_var_name
| {GLOBAL | @@GLOBAL.} system_var_name
| [SESSION | @@SESSION. | @@] system_var_name
}
```



Precautions
-------------------------

When using SET GLOBAL to set a global variable, PolarDB-X will persist it, and it will still take effect after the instance is restarted. In addition, after SET GLOBAL is set successfully, all existing connections will take effect.

example
-----------------------

```sql
mysql> SET @foo='bar';
Query OK, 0 rows affected (0.00 sec)
mysql> SELECT @foo;
+------+
| @foo |
+------+
| bar  |
+------+
1 row in set (0.01 sec)
mysql> SET @@time_zone='+09:00';
Query OK, 0 rows affected (0.01 sec)
mysql> SELECT @@time_zone;
+-------------+
| @@time_zone |
+-------------+
| +09:00      |
+-------------+
1 row in set (0.00 sec)
mysql> SET GLOBAL time_zone='+09:00';
Query OK, 0 rows affected (0.04 sec)
mysql> SELECT @@GLOBAL.time_zone;
+--------------------+
| @@global.time_zone |
+--------------------+
| +09:00             |
+--------------------+
1 row in set (0.02 sec)
```


