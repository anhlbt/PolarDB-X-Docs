Function Introduction (WIP)
=========================

PolarDB-X has newly added the separation of three rights mode. You can assign the rights of high-privilege accounts to the three roles of system administrator, security administrator, and audit administrator to avoid risks caused by highly concentrated permissions and enhance database security. safety.

Risks and Solutions
----------------------------

* **Risk**

In the traditional database operation and maintenance mode, the authority of the database administrator DBA (Database Administrator) is too high and centralized, and it is easy to bring risks to the business in certain scenarios:
* DBA misjudgments lead to system security incidents.

* DBA conducts illegal operations for some purpose.

* DBAs, third-party outsourcers or program developers have unauthorized access to sensitive information.





* **solution**

PolarDB-X newly supports the separation of three rights model, improving the independent control system in which DBAs exercise privileges in traditional database operation and maintenance, enabling database administrators DBA, security administrators DSA (Department Security Administrator) and audit administrators DAA (Data Audit Administrator) The rights and responsibilities of the three parties are more clearly defined. in:
* Database administrator (DBA): only have DDL (Data Definition Language) authority.

* Security administrator (DSA): only has the authority to manage roles (Role) or users (User) and grant permissions to other accounts.

* Audit Administrator (DAA): only has the authority to view audit logs.







Comparison of permissions of database system accounts
---------------------------------

The following table shows the comparison of permissions of different database system accounts in the default mode and the separation of powers mode.

**illustrate**

* The high-privilege account in the default mode is the system administrator account. For more details about high-privilege accounts, please refer to [Account Type](https://help.aliyun.com/document_detail/172163.htm#title-1px-w9q-n64).

* Turning on or off the separation of powers mode only affects the permissions of the system account (that is, the high-privilege account, the system administrator account, the security administrator account, and the audit administrator account), and the normal account permissions are not affected by the mode change.

* In the separation of powers mode, although all system accounts do not have DML (Data Manipulation Language), DQL (Data Query Language) or DAL (Data Administration Language) permissions, the security administrator can still grant these permissions to ordinary accounts.

* ✔️ in the table means that you have the permission, and ❌ means you don’t have the permission.


| Permission classification | Description | Default mode (high-privilege account) | Three-power separation mode (system administrator account) | Three-power separation mode (security administrator account) | Three-power separation mode (audit administrator account) |
|---------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------|---------|---------|---------|
| DDL     | * ALTER TABLE  <br />* CREATE TABLE  <br />* CREATE VIEW  <br />* CREATE INDEX  <br />* CREATE CCL_RULE  <br />* DROP VIEW  <br />* DROP INDEX  <br />* DROP TABLE  <br />* TRUNCATE TABLE | ✔️    | ✔️      | ❌       | ❌       |
| DML     | * DELETE  <br />* UPDATE  <br />* INSERT                                                                                                                                                                                                                                                                                                                 | ✔️    | ❌       | ❌       | ❌       |
| DQL     | * SELECT  <br />* EXPLAIN                                                                                                                                                                                                                                                                                                                                                                  | ✔️    | ❌       | ❌       | ❌       |
| DAL     | * SHOW CCL_RULE  <br />* SHOW INDEX                                                                                                                                                                                                                                                                                                                                                        | ✔️    | ❌       | ❌       | ❌       |
| Account or role-related | [Account Authority Management](account.md)<br />[Role Authority Management](role.md) | ✔️ | ❌ | ✔️ | ❌ |
| View audit log | View audit log information in the following two tables: <br />* `information_schema.polardbx_audit_log` <br />* `information_schema.polardbx_ddl_log` | ✔️ | ❌ | ❌ | ✔️ |



usage restrictions
-------------------------

System accounts (including system administrator accounts, security administrator accounts, and audit administrator accounts) under the separation of powers mode have the following restrictions:

* The GRANT ROLE or REVOKE ROLE commands for system accounts are not supported.

* The GRANT PRIVILEGES or REVOKE PRIVILEGES commands for system accounts are not supported.

* The password of the system account can only be modified by the corresponding account. For example, the password of the system administrator account can only be modified by the system administrator account, and cannot be modified by other accounts.

* System accounts do not support the SET DEFAULT ROLE command.



