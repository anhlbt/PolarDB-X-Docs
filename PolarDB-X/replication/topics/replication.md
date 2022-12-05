Replication
================================



characteristic
-----------------------

* Compatible with MySQL Binlog file format and Dump protocol. The global Binlog of PolarDB-X is generated based on the physical Binlog of the DN node, which eliminates the details of distributed transactions and only retains the characteristics of stand-alone transactions. At the same time, the global Binlog is compatible with the MySQL Binlog file format, and the data subscription method is also fully compatible with the MySQL Dump protocol. You can subscribe to the transaction log of PolarDB-X just like using a stand-alone MySQL.

* Guarantee the integrity and order of distributed transactions. The global Binlog does not simply aggregate physical Binlogs together, but ensures the integrity and order of distributed transactions through merging and sorting, thereby achieving high data consistency. For example, in the transfer scenario, based on the global Binlog capability, the downstream MySQL connected to PolarDB-X can query the consistent balance at any time.

* Provide 7x24 hours service capability, simple operation and maintenance. The global Binlog eliminates the internal details of PolarDB-X (you can regard PolarDB-X as a stand-alone MySQL at this time) to avoid the impact of changes in the instance on the data subscription link. PolarDB-X guarantees the service capability of the global Binlog through a series of protocols and algorithms, ensuring that various changes within the instance (such as HA switching, adding and deleting nodes, executing Scale Out or distributed DDL, etc.) will not affect the data subscription chain normal work of the road.




usage restrictions
-------------------------

* The data subscription method in the Gtid (Global Transaction Identifier) ​​mode is not currently supported.

* The merging of distributed transactions is only supported when the transaction strategy is specified as TSO (that is, a higher-strength consistency guarantee).




SQL statements supported by the data feed source
-----------------------------------

**Description** To execute the following SQL statements, SUPER or REPLICATION CLIENT authority is required. For permission operations, please refer to [Account and Permission System](../../maintance/topics/account.md).

* View PolarDB-X global Binlog file list.

```sql
SHOW BINARY LOGS
```


* View the Binlog information of PolarDB-X as the master role.

```sql
SHOW MASTER STATUS
```


* View specific event information in the global Binlog file.

```sql
SHOW BINLOG EVENTS
[IN 'log_name']
[FROM pos]
[LIMIT [offset,] row_count]
```



SQL statements supported by the data subscription target (WIP)
------------------------------------

If the data subscription target is standard MySQL, the MySQL Replicate command is currently supported.

* Set the source data source information that needs to be synchronized at the data subscription target end.

```sql
CHANGE MASTER TO option [, option] ... [ channel_option ]
option: {
MASTER_BIND = 'interface_name'
| MASTER_HOST = 'host_name'
| MASTER_USER = 'user_name'
| MASTER_PASSWORD = 'password'
| MASTER_PORT = port_num
| PRIVILEGE_CHECKS_USER = {'account' | NULL}
| REQUIRE_ROW_FORMAT = {0|1}
| REQUIRE_TABLE_PRIMARY_KEY_CHECK = {STREAM | ON | OFF}
| ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS = {OFF | LOCAL | uuid}
| MASTER_LOG_FILE = 'source_log_name'
| MASTER_LOG_POS = source_log_pos
| MASTER_AUTO_POSITION = {0|1}
| RELAY_LOG_FILE = 'relay_log_name'
| RELAY_LOG_POS = relay_log_pos
| MASTER_HEARTBEAT_PERIOD = interval
| MASTER_CONNECT_RETRY = interval
| MASTER_RETRY_COUNT = count
| SOURCE_CONNECTION_AUTO_FAILOVER = {0|1}
| MASTER_DELAY = interval
| MASTER_COMPRESSION_ALGORITHMS = 'value'
| MASTER_ZSTD_COMPRESSION_LEVEL = level
| MASTER_SSL = {0|1}
| MASTER_SSL_CA = 'ca_file_name'
| MASTER_SSL_CAPATH = 'ca_directory_name'
| MASTER_SSL_CERT = 'cert_file_name'
| MASTER_SSL_CRL = 'crl_file_name'
| MASTER_SSL_CRLPATH = 'crl_directory_name'
| MASTER_SSL_KEY = 'key_file_name'
| MASTER_SSL_CIPHER = 'cipher_list'
| MASTER_SSL_VERIFY_SERVER_CERT = {0|1}
| MASTER_TLS_VERSION = 'protocol_list'
| MASTER_TLS_CIPHERSUITES = 'ciphersuite_list'
| MASTER_PUBLIC_KEY_PATH = 'key_file_name'
| GET_MASTER_PUBLIC_KEY = {0|1}
| NETWORK_NAMESPACE = 'namespace'
| IGNORE_SERVER_IDS = (server_id_list)
}
channel_option:
FOR CHANNEL channel
server_id_list:
[server_id [, server_id] ... ]
```



* Enable master-standby synchronization

```sql
START {SLAVE | REPLICA}
```



* Stop master-standby synchronization

```sql
STOP {SLAVE | REPLICA}
```



* To reset the master-standby synchronization, you need to stop the master-standby synchronization first

```sql
RESET {SLAVE | REPLICA} [ALL] [channel_option]
channel_option:
FOR CHANNEL channel
```


**Note** If the target end is PolarDB-X, the relevant Replicate command is currently not supported.




