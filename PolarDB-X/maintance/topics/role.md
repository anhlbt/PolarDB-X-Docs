Role permission management
===========================

This article introduces syntax-level examples related to role permission management.

PolarDB-X is compatible with native MySQL 8.0 role-based access control, please refer to [Role-based Access Control](https://dev.mysql.com/doc/refman/8.0/en/roles.html).

Creating a Role
-------------------------

grammar:

```sql
CREATE ROLE role [, role]...
```



Like user, role also consists of Name and Host, among which:

* Name cannot be empty;

* Host must meet the following rules:
* Must be a pure IP address, which can contain underscore (_) and percent sign (%), but these two symbols only represent 2 ordinary characters and do not have the meaning of wildcards;

* Host blank is equal to %, but it is also an exact match and does not have the meaning of wildcard.







Example:

```sql
mysql> CREATE ROLE 'role_ro'@'%', 'role_write';
```



delete role
-------------------------

grammar:

```sql
DROP ROLE role [, role] ...
```



Example:

```sql
mysql> DROP ROLE 'role_ro'@'%';
```



grant role
-------------------------

**Grant permission to role**

grammar:

```sql
GRANT priv_type [, priv_type] ... ON priv_level TO role [, role]... [WITH GRANT OPTION]
```



Example:

```sql
mysql> GRANT ALL PRIVILEGES ON db1.* TO 'role_write';
```

**Grant role to user**

grammar:

```sql
GRANT role [, role] ...
TO user_or_role [, user_or_role] ...
[WITH ADMIN OPTION]
```



illustrate:

* To execute this command, one of the following conditions must be met:
* The current user has CREATE_USER permission;

* The current user has admin permission for Role;



* If the WITH ADMIN OPTION option is included, the target user has admin permission for the Role;

* Granting a role to a user does not mean that the user already has the permissions under the role, you also need to set the role that needs to be activated for the user through the `SET DEFAULT ROLE` statement and the `SET ROLE` statement.




Example:

```sql
mysql> GRANT 'role_write' TO 'user1'@'127.0.0.1';
```

**SET DEFAULT ROLE**

grammar:

```sql
SET DEFAULT ROLE
{NONE | ALL | role [, role ] ...}
TO user [, user ] ...
```



One of the following conditions must be met to execute this command:

* The Role mentioned in the statement has been granted to the target user through the GRANT command;

* The current user is the target user, or the current user has CREATE_USER permission.




Example:

```sql
mysql> SET DEFAULT ROLE 'role_write' TO 'user1'@'127.0.0.1';
```

**Set the current connection role**

grammar:

```sql
SET ROLE {
DEFAULT
| NONE
| ALL
| ALL EXCEPT role [, role ] ...
| role [, role ] ...
}
```


**illustrate**

* If you choose to execute `SET ROLE DEFAULT`, the currently activated role is the role selected in the `SET DEFAULT ROLE` command;

* The role activated by this syntax is only valid for the user using the current connection.




Example:

```sql
mysql> SET ROLE 'role_write';;
```



View role permissions
---------------------------

grammar:

```sql
SHOW GRANTS
[FOR user_or_role
[USING role [, role] ...]]
```



Example:

```sql
mysql>  SHOW GRANTS FOR 'role_write'@'%';
+---------------------------------------------------+
| GRANTS FOR 'ROLE_WRITE'@'%'                       |
+---------------------------------------------------+
| GRANT USAGE ON *.* TO 'role_write'@'%'            |
| GRANT ALL PRIVILEGES ON db1.* TO 'role_write'@'%' |
+---------------------------------------------------+
mysql> SHOW GRANTS FOR 'user1'@'127.0.0.1' USING 'role_write';
+------------------------------------------------------+
| GRANTS FOR 'USER1'@'127.0.0.1'                       |
+------------------------------------------------------+
| GRANT USAGE ON *.* TO 'user1'@'127.0.0.1'            |
| GRANT ALL PRIVILEGES ON db1.* TO 'user1'@'127.0.0.1' |
| GRANT 'role_write'@'%' TO 'user1'@'127.0.0.1'        |
+------------------------------------------------------+
-- Execute as user1's session
mysql> SELECT CURRENT_ROLE();
+------------------+
| CURRENT_ROLE()   |
+------------------+
| 'role_write'@'%' |
+------------------+
```



recycle role
-------------------------

**Recycle role permissions**

grammar:

```sql
REVOKE priv_type [, priv_type] ... ON priv_level FROM role [, role]...
```



Example:

```sql
mysql> REVOKE ALL PRIVILEGES ON db1.* FROM 'role_write';
mysql> SHOW GRANTS FOR 'role_write'@'%';
+----------------------------------------+
| GRANTS FOR 'ROLE_WRITE'@'%'            |
+----------------------------------------+
| GRANT USAGE ON *.* TO 'role_write'@'%' |
+----------------------------------------+
```

**Recycle user's permissions**

grammar:

```sql
REVOKE role [, role ] ... FROM user_or_role [, user_or_role ] ...
```



Example:

```sql
mysql> SHOW GRANTS FOR 'user1'@'127.0.0.1';
+-----------------------------------------------+
| GRANTS FOR 'USER1'@'127.0.0.1'                |
+-----------------------------------------------+
| GRANT USAGE ON *.* TO 'user1'@'127.0.0.1'     |
| GRANT SELECT ON db1.* TO 'user1'@'127.0.0.1'  |
| GRANT 'role_write'@'%' TO 'user1'@'127.0.0.1' |
+-----------------------------------------------+
mysql> REVOKE 'role_write' FROM 'user1'@'127.0.0.1';
mysql> SHOW GRANTS FOR 'user1'@'127.0.0.1';
+----------------------------------------------+
| GRANTS FOR 'USER1'@'127.0.0.1'               |
+----------------------------------------------+
| GRANT USAGE ON *.* TO 'user1'@'127.0.0.1'    |
| GRANT SELECT ON db1.* TO 'user1'@'127.0.0.1' |
+----------------------------------------------+
```

