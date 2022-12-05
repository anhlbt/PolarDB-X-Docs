Use mysqldump to import and export data
======================================

This article introduces several common scenarios and detailed operation steps for importing and exporting PolarDB-X data through the mysqldump tool.

PolarDB-X supports MySQL official data export tool mysqldump. For a detailed description of the mysqldump command, see [MySQL Official Documentation](http://dev.mysql.com/doc/refman/5.6/en/mysqldump.html).

**Description** mysqldump is suitable for offline import and export of small data volume (less than 10 million). If you need to complete a larger amount of data or real-time data migration tasks, it is recommended to use DTS for data migration.

Introduction to mysqldump tool
----------------------------------

mysqldump can export table structure information and table data, and convert them into SQL statement format for users to import directly. The SQL syntax is as follows:

```sql
DROP TABLE IF EXISTS `table_name`;
CREATE TABLE `table_name` (
`id` int(11) NOT NULL,
`k` int(11) NOT NULL DEFAULT '0',
...
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4  dbpartition by hash(`id`);
INSERT INTO `table_name` VALUES (...),(...),...;
INSERT INTO `table_name` VALUES (...),(...),...;
...
```



Example of how to use the command to export data with the mysqldump tool:

```shell
shell> mysqldump -h ip -P port -u user -pPassword --default-character-set=char-set --net_buffer_length=10240 --no-create-db --no-create-info --skip-add-locks --skip-lock-tables --skip-tz-utc --set-charset  [--hex-blob] [--no-data] database [table1 table2 table3...] > dump.sql
```



The parameter description of mysqldump can be viewed or queried through the `mysqldump --help` command [MySQL official document](http://dev.mysql.com/doc/refman/5.6/en/mysqldump.html), the common parameters are described as follows, Please enter according to the actual situation:


| parameter name | description |
|---------------------|---------------------------------------------------------------------|
| ip | The IP of the PolarDB-X instance. |
| port | The port of the PolarDB-X instance. |
| user | The user name of PolarDB-X. |
| password | The password of PolarDB-X, note that there is a -p in front, and there is no space between them. |
| char-set | The specified encoding. |
| --hex-blob | Export binary string fields in hexadecimal format. This option must be used if there is binary data. Affected field types include BINARY, VARBINARY, BLOB. |
| --no-data | Do not export data. |
| table | Specifies a table to export. By default, all tables in the database are exported. |
| --no-create-info | Do not export table creation information |
| --net_buffer_length | Transmission buffer size. Affects the length of the Insert statement, the default value is 1046528. |



There are two ways to import the exported SQL statement format file into the database:


* SOURCE statement imports data

```shell
## 1. Login to the database
shell> mysql -h ip -P port -u user -pPassword --default-character-set=char-set
## 2. Import data by executing the sql statement in the file through the source statement
mysql> source dump.sql
```



* mysql command to import data

```shell
shell> mysql -h ip -P port -u user -pPassword --default-character-set=char-set< /yourpath/dump.sql
```






The following describes the usage examples of the mysqldump tool from different scenarios.

It is not recommended to export the table structure when transferring data between PolarDB-X and MySQL, because PolarDB-X includes the function of sub-database and sub-table, [CREATE TABLE](../../dev-guide/topics/create-table.md) The split function in is not compatible with MySQL, and the incompatible keywords include:


* DBPARTITION BY hash(partition_key)

* TBPARTITION BY hash(partition_key)

* TBPARTITIONS N

* BROADCAST




If you export the table structure, you need to modify the table creation statement in the exported SQL statement file to import it correctly. Therefore, it is recommended to only export the data in the table, manually log in to the database to create the table, and then import the data.

Scenario 1: Import from MySQL to PolarDB-X
----------------------------------------

To import data from MySQL to PolarDB-X, please follow the steps below.

1. Export data from MySQL to a file. Enter the following command to export the data in the table from MySQL (exporting the table structure is not recommended), assuming that the exported file is dump.sql.

```shell
mysqldump -h ip -P port -u user -pPassword --default-character-set=char-set --net_buffer_length=204800 --no-create-db --no-create-info --skip-add-locks --skip-lock-tables --skip-tz-utc --set-charset --hex-blob database [table1 table2 table3...] > dump.sql
```



2. Log in to PolarDB-X and manually create the target table. For the syntax of PolarDB-X table creation statements, please refer to [CREATE TABLE](../../dev-guide/topics/create-table.md). If the --no-create-info parameter is not added, the exported dump.sql file contains the table creation statement on the MySQL side, and the table creation statement can also be modified in the file.

3. Import the data file into PolarDB-X. You can import data files to PolarDB-X in the following two ways:
* Log in to the target PolarDB-X through the `mysql -h ip -P port -u user -pPassword --default-character-set=char-set` command, and execute the `source /yourpath/dump.sql` command to import data to the target PolarDB-X.

* Directly import data to the target PolarDB-X through `mysql -h ip -P port -u user -pPassword --default-character-set=char-set< /yourpath/dump.sql` command.



**illustrate**
* `default-character-set` in the above two commands should be set to the actual data encoding. If it is a Windows platform, the file path specified by the source command needs to escape the delimiter.

* The first method will echo all the steps to the screen, the speed is slightly slower, but you can observe the import process.

* When importing, due to the difference in the implementation of some PolarDB-X and MySQL, an error may be reported, and the error message is similar to `ERROR 1231 (HY000): [a29ef6461c00000][10.117.207.130:3306][****]Variable @ saved_cs_client can't be set to the value of @@character_set_client`. Such error messages do not affect the correctness of the imported data.








Scenario 2: Import from one PolarDB-X to another PolarDB-X
-------------------------------------------------

Suppose you have a PolarDB-X in the test environment before, and after the test, you need to import some table structures and data in the test process to PolarDB-X in the production environment, then you can follow the steps below.

1. Export the data from the source PolarDB-X to a text file. For details, see Step 1 of Scenario 1.

2. Import the data file to PolarDB-X. Please refer to Step 3 of Scenario 1.

3. Manually create a Sequence object. mysqldump does not export the Sequence object in PolarDB-X, so if the Sequence object is used in the source PolarDB-X and needs to continue to use the same Sequence object in the target PolarDB-X, you need to manually add the Sequence object in the target PolarDB-X Create a Sequence object with the same name. Specific steps are as follows:


1. Execute SHOW SEQUENCES on the source PolarDB-X to get the status of the Sequence object in the current PolarDB-X.

2. Use the CREATE SEQUENCE command to create a new Sequence object on the target PolarDB-X database.




For details about the Sequence command, see [Sequence](../../dev-guide/topics/sequence.md).





Scenario 3: Export data from PolarDB-X to MySQL
------------------------------------------

The process of exporting data from PolarDB-X to MySQL is similar to the process of importing data between PolarDB-X, and it is also divided into the following steps.


1. Export the data from the source PolarDB-X to a text file. For details, see Step 1 of Scenario 1.

2. Log in to MySQL and manually create the target table. If the exported data contains table creation statements, you need to modify the table creation statements in the exported file, and delete the keywords in PolarDB-X that are not compatible with MySQL and other information.

For example a split table in PolarDB-X:

```sql
CREATE TABLE `table_name` (
`id` int(11) NOT NULL,
`k` int(11) NOT NULL DEFAULT '0',
...
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4 dbpartition by hash(`id`);
```



The MySQL incompatible split function statement needs to be removed and changed to:

```sql
CREATE TABLE `table_name` (
`id` int(11) NOT NULL,
`k` int(11) NOT NULL DEFAULT '0',
...
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4;
```



3. Import the data file to PolarDB-X. Please refer to Step 3 of Scenario 1.




