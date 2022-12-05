Use Batch Tool to import and export data
=========================================

This article describes how to import and export data through the Batch Tool.

Tool introduction
-------------------------

The Batch Tool tool is a tool developed by the PolarDB-X team to provide data import and export services for the PolarDB-X database.

The Batch Tool tool is implemented with JAVA statements. The core is the producer-consumer model, which supports multi-threaded operations and provides functions such as batch export, batch import, batch deletion, and batch update. Data is transmitted in csv file format, which is convenient for users to interact with data.

The command usage and parameters of the Batch Tool are as follows:

```shell
usage: BatchTool [-?] [-batchsize <arg>] [-con <consumer count>] [-cs
<charset>] -D <database> [-dir <directory>] [-f <from>] [-F
<filenum>] [-func] -h <host> [-header] [-i] [-in] [-L <line>]
[-lastSep] [-lb] [-local] [-maxConn <arg>] [-minConn <arg>]
[-noesc] [-np] [-O <order by type>] -o <operation> [-OC <ordered
column>] -p <password> [-P <port>] [-para] [-pre <prefix>] [-pro
<producer count>] [-quote <auto/force/none>] [-readsize <arg>]
[-rfonly] [-ringsize <arg>] -s <sep> [-t <table>] -u <user> [-w
<where>]
-?,--help                              Help message.
-batchsize,--batchSize <arg>           Batch size of emitted tuples.
-con,--consumer <consumer count>       Configure number of consumer
threads.
-cs,--charset <charset>                Define charset of files.
-D,--database <database>               Database to use.
-dir,--dir <directory>                 Directory path including files to
import.
-f,--from <from>                       Source file(s), separated by ; .
-F,--filenum <filenum>                 Fixed number of exported files.
-func,--sqlfunc                        Use sql function to update.
-h,--host <host>                       Connect to host.
-header,--header                       Whether the header line is column
names.
-H,--historyFile <filename>            history file name for resuming from breakpoint
-i,--ignoreandresume                   Flag of insert ignore & resume from breakpoint
-in,--wherein                          Using where ... in (...)
-L,--line <line>                       Max line limit of exported files.
-lastSep,--withLastSep                 Whether line ends with separator.
-lb,--loadbalance                      If using load balance.
-local,--localmerge                    o local merge sort.
-maxConn,--maxConnection <arg>         Max connection number limit.
-minConn,--minConnection <arg>         Mim connection number limit.
-noesc,--noescape                      Don't escape values.
-np,--partition No use of partition.
-O,--orderby <order by type>           asc or desc.
-o,--operation <operation>             Batch operation type: export /
import / delete / update.
-OC,--orderCol <ordered column>        col1;col2;col3.
-p,--password <password>               Password to use when connecting to
server.
-P,--port <port>                       Port number to use for connection.
-para,--paraMerge                      Using parallel merge when doing
order by export.
-pre,--prefix <prefix>                 Export file name prefix.
-pro,--producer <producer count>       Configure number of producer
threads.
-quote,--quoteMode <auto/force/none>   The mode of how field values are
enclosed by double-quotes when
exporting table. Default value is
auto.
-readsize,--readSize <arg>             Read block size in MB.
-rfonly,--rfonly                       Only read and process file, no sql
execution.
-ringsize,--ringBufferSize <arg>       Ring buffer size.
-s,--sep <sep>                         Separator between fields
(delimiter).
-t,--table <table>                     Target table.
-tps,--tpsLimit <arg>                  Tps limit
-u,--user <user>                       User for login.
-w,--where <where>                     Where condition: col1>99 AND
No<100 ...
```

**Parameter Description**

The common parameters are described as follows:

* -o: batch operation, including export, import, delete, update four options.

* -t: Specify the target table name, which can only be a single table.

* -s: Specifies the separator, which can be a character or a string.

* -f: Specify the source file, use a semicolon ";" to separate multiple file names.

* -OC: Specifies the column name used for sorting when exporting, and multiple columns are separated by semicolons ";".

* -cs: Specify the character set of the text file, the default is utf-8.

* -lastSep: Whether each line of the file ends with a separator.

* -quote: Specify the quotation mark surround mode when exporting or importing, including the following three optional values:
* auto: default mode, double quotes will be added according to whether the field value contains special characters (such as separators, newlines, etc.);

* force: Force each field value to add double quotes;

* none: Forcibly do not add double quotes (applicable to situations where the known table field types are all numeric, or string fields do not contain special characters).




* -header: Whether the first line is a field name.

* -i: Whether to enable insert ignore and resumable upload.

* -pre: Specifies the prefix of the exported file name.

* -F: Specifies the number of exported files.




tool acquisition
-------------------------

The jar package of Batch Tool, click to download: [Batch_tool tool](http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/attach/183466/cn_zh/1633679163870/batch-tool% 20%281%29.jar).

Example of use
-------------------------

Take the compiled batch-tool.jar as an example, see the parameter description:

```shell
java -jar batch-tool.jar -?
```



* Batch export data

```shell
## 1. Default export (the number of files is equal to the number of fragments of the table)
java -jar batch-tool.jar -h 127.0.0.1 -u testUser -p testPasswd -P 3306 -D db_name -o export -t table_name -s ,
## 2. Number of exported files=3 (-F: specify the number of files)
java -jar batch-tool.jar -h 127.0.0.1 -u testUser -p testPasswd -P 3306 -D db_name -o export -t table_name -s , -F 3
## 3. Specify the maximum number of lines in a single file=10000 (-L: specify the number of lines in a single file)
java -jar batch-tool.jar -h 127.0.0.1 -u testUser -p testPasswd -P 3306 -D db_name -o export -t table_name -s , -L 10000
## 4. With where condition If the condition has spaces, you need to use quotation marks (-w: where condition statement)
java -jar batch-tool.jar -h 127.0.0.1 -u testUser -p testPasswd -P 3306 -D db_name -o export -t table_name -s , -w "id < 150 and id > 120"
```



* Import data in batches (you need to manually create the target table, Batch Tool only includes data transfer)

```shell
## 1. Separate multiple files with a semicolon (;)
java -jar batch-tool.jar -h 127.0.0.1 -u testUser -p testPasswd -P 3306 -D db_name -o import -t table_name -s , -f "table_name_0;table_name_1;table_name_2;table_name_3"
## 2. By default, sharding is inserted according to the split key. If not used, just turn on the -np switch
java -jar batch-tool.jar -h 127.0.0.1 -u testUser -p testPasswd -P 3306 -D db_name -o import -t table_name -s , -f "file0;file2" -np
## 3. Specify producer and consumer threads (-pro: producer thread, read file thread; -con: consumer thread, import data thread)
java -jar batch-tool.jar -h 127.0.0.1 -u testUser -p testPasswd -P 3306 -D db_name -o import -t table_name -s , -f "table_name_0;table_name_1" -np -pro 16 -con 16
## 4. Open insert ignore and breakpoint resume
java -jar batch-tool.jar -h 127.0.0.1 -u testUser -p testPasswd -P 3306 -D db_name -o import -t table_name -s , -f "table_name_0;table_name_1" -i
```



* Delete data in batches (delete the data in the file contained in the database, principle: build a DELETE statement, fill the data in the file according to the table structure)

```shell
java -jar batch-tool.jar -h 127.0.0.1 -u testUser -p testPasswd -P 3306 -D db_name -o delete -t table_name -s , -f "file0"
```



* Batch update data (update the data in the file contained in the database, principle: construct UPDTATE statement, fill the data in the file according to the table structure)

```shell
java -jar batch-tool.jar -h 127.0.0.1 -u testUser -p testPasswd -P 3306 -D db_name -o update -t table_name -s , -f "file0"
```





