Account authority management
===========================

This article introduces the related operations of account permission management.

The usage of the PolarDB-X account and permission system is consistent with MySQL 5.7, and supports statements such as GRANT, REVOKE, SHOW GRANTS, CREATE USER, DROP USER, SET PASSWORD, etc. Currently, it supports the granting of library-level and table-level permissions, global-level and column-level permissions Currently not supported.

Create an account
-------------------------

grammar:

```sql
CREATE USER [IF NOT EXISTS] user IDENTIFIED BY 'password';
```



Among them, user determines an account through the combination of user name and host name `'username'@'host'`, and the account rules are as follows:

* username is the created username, and the username follows the following rules;
* Case Sensitive;

* The length must be greater than or equal to 4 characters and less than or equal to 20 characters;

* must start with a letter;

* Characters can include uppercase letters, lowercase letters, numbers.




* host specifies the host on which the created user can log in. The same user name but different host names also represent different accounts, and the following rules must be met:
* HOST must be a pure IP address, which can contain _ and % wildcards (_ represents one character, % represents 0 or more characters). HOSTs containing wildcards need to be enclosed in single quotes, such as lily@'0.9.%.%', david@'%';

* Assuming that there are two users in the system who both match the user who is currently ready to log in, the user with the longest prefix match (the longest IP segment excluding wildcards) shall prevail. For example, the system has two users david@'30.9.12_.234' and david@'30.9.1%.234'. If you log in to david on the host 30.9.127.234, you will use david@'30.9.12_.234' user;

* When VPC is enabled, the IP address of the host will change. To avoid invalid configuration in the account and permission system, please configure HOST as '%' to match any IP.




* password is the user password, which must meet the following rules:
* The length must be greater than or equal to 6 characters and less than or equal to 20 characters;

* Characters can include uppercase letters, lowercase letters, numbers, special characters (@#$%\^\&+=).







Example:

```sql
mysql> CREATE USER 'user1'@'127.0.0.1' IDENTIFIED BY '123456';
mysql> CREATE USER IF NOT EXISTS 'user2'@'%' identified by '123456';
```



Change account password
---------------------------

grammar:

```sql
SET PASSWORD FOR user = PASSWORD('auth_string')
```



Example:

```sql
mysql> SET PASSWORD FOR 'user1'@'127.0.0.1' = PASSWORD('654321');
```



delete account
-------------------------

grammar:

```sql
DROP USER user;
```



Example:

```sql
mysql> DROP USER 'user2'@'%';
```



Grant account permissions
---------------------------

grammar:

```sql
GRANT privileges ON database.table TO user;
```



Among them, privileges is a specific permission type, and the database permission levels from high to low are: global level permissions (not supported for now), database level permissions, table level permissions, and column level permissions. PolarDB-X currently supports 8 basic permission items associated with tables: CREATE, DROP, ALTER, INDEX, INSERT, DELETE, UPDATE, SELECT.

* TRUNCATE operation requires DROP permission on the table;

* REPLACE operation requires INSERT and DELETE permissions on the table;

* CREATE INDEX and DROP INDEX operations require INDEX permission on the table;

* CREATE SEQUENCE requires database-level create table (CREATE) permission;

* DROP SEQUENCE requires database-level delete table (DROP) permission;

* ALTER SEQUENCE requires database-level alter table (ALTER) permission;

* The INSERT ON DUPLICATE UPDATE statement requires INSERT and UPDATE permissions on the table.




Example:

```sql
mysql> GRANT SELECT,UPDATE ON `db1`.* TO 'user1'@'127.0.0.1';
```



View account permissions
---------------------------

grammar:

```sql
SHOW GRANTS [FOR user];
```



You can use current_user() to get the current user.

Example:

```sql
mysql> SHOW GRANTS FOR 'user1'@'127.0.0.1';
+------------------------------------------------------+
| GRANTS FOR 'USER1'@'127.0.0.1'                       |
+------------------------------------------------------+
| GRANT USAGE ON *.* TO 'user1'@'127.0.0.1'            |
| GRANT SELECT, UPDATE ON db1.* TO 'user1'@'127.0.0.1' |
+------------------------------------------------------+
mysql> SHOW GRANTS FOR current_user();
+------------------------------------------------------+
| GRANTS FOR 'USER1'@'127.0.0.1'                       |
+------------------------------------------------------+
| GRANT USAGE ON *.* TO 'user1'@'127.0.0.1'            |
| GRANT SELECT, UPDATE ON db1.* TO 'user1'@'127.0.0.1' |
+------------------------------------------------------+
```



Reclaim Account Permissions
---------------------------

grammar:

```sql
REVOKE privileges ON database.table TO user;
```



Example:

```sql
mysql> REVOKE UPDATE ON db1.* FROM 'user1'@'127.0.0.1';
mysql> SHOW GRANTS FOR 'user1'@'127.0.0.1';
+----------------------------------------------+
| GRANTS FOR 'USER1'@'127.0.0.1'               |
+----------------------------------------------+
| GRANT USAGE ON *.* TO 'user1'@'127.0.0.1'    |
| GRANT SELECT ON db1.* TO 'user1'@'127.0.0.1' |
+----------------------------------------------+
```


