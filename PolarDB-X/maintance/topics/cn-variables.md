Calculation layer variables
==========================

PolarDB-X is a storage-computing separation architecture. Common CN variables are listed here (some of which are configurable in the console).


| Variable name | Restart or not | Default value | Scope | Remarks |
|----------------------------------------------|-------------------|-----------------|---------------------------------------------------------------------|------------------------------------------------------------------|
| PLAN_CACHE | NO | TRUE | [TURE/FALSE] | Switch for plan caching |
| ENABLE_RECYCLEBIN | No | FALSE | [TURE/FALSE] | Turn on the recycle bin switch |
| SHOW_TABLES_CACHE | No | FALSE | [TURE/FALSE] | Whether to cache the result of show tables |
| MERGE_CONCURRENT | No | FALSE | [TURE/FALSE] | Full parallel execution switch, mainly affects the parallel execution degree of simple queries of DDL and full pushdown |
| MERGE_UNION | No | FALSE | [TURE/FALSE] | Disabled by default, enabling means that the physical SQL does not perform union optimization, and the pushed-down physical SQL will be executed serially |
| MERGE_UNION_SIZE | No | -1 | [0-10000] | The number of physical SQL merged by union, and the default is derived adaptively based on the number of available connections in the connection pool |
| TABLE_META_CACHE_EXPIRE_TIME | No | 300 | [0-180000] | Metadata cache expiration time |
| COLUMN_LABEL_INSENSITIVE | No | TRUE | [FALSE/TRUE] | Returns whether the column is case sensitive |
| RECORD_SQL | NO | TRUE | [FALSE/TRUE] | Audit log switch |
| SOCKET_TIMEOUT | No | 900000 | [0~3600000] | Physical SQL timeout |
| TRANSACTION_POLICY | No | TSO | [XA/TSO/TSO_READONLY] | Transaction Policy |
| SHARE_READ_VIEW | NO | FALSE | [TRUE/FALSE] | Share ReadView switch |
| ENABLE_TRX_SINGLE_SHARD_OPTIMIZATION | No | TRUE | [TRUE/FALSE] | Transaction single shard optimization switch |
| GET_TSO_TIMEOUT | No | 10 | [1-1800] | Get TSO timestamp timeout |
| MAX_TRX_DURATION | No | 28800 | [1-180000] | Transaction physical timeout |
| TRANSACTION_ISOLATION | No | REPEATABLE_READ | [READ_UNCOMMITTED/<br />READ_COMMITTED/<br />REPEATABLE_READ/<br />SERIALIZABLE] | Transaction isolation level |
| GROUP_CONCURRENT_BLOCK | No | TRUE | [TRUE/FALSE] | Sub-database level execution strategy in non-MPP mode |
| SEQUENTIAL_CONCURRENT_POLICY | No | FALSE | [TRUE/FALSE] | Concurrent execution policy for orders in non-MPP mode |
| DML_SKIP_DUPLICATE_CHECK_FOR_PK | No | TRUE | [TRUE/FALSE] | Whether to skip primary key conflict check during DML |
| DML_SKIP_CRUCIAL_ERR_CHECK | No | FALSE | [TRUE/FALSE] | Whether to allow transactions with DML errors to continue to be submitted during the DML process |
| DML_USE_RETURNING | No | TRUE | [TRUE/FALSE] | Whether to use returning optimization |
| BROADCAST_DML | No | FALSE | [TRUE/FALSE] | Whether to allow the writing of the broadcast table without going through the distributed transaction |
| SEQUENCE_STEP | No | 10000 | [1-10000000] | SEQUENCE step size, the default is 100,000 |
| MERGE_DDL_TIMEOUT | No | 0 | [1-10000000] | DDL physical connection timeout, the default is 0, no timeout |
| MERGE_DDL_CONCURRENT | No | FALSE | [FALSE/TRUE] | Whether ddl adopts full parallel mode, default library-level concurrency |
| SLOW_SQL_TIME | No | 1000 | [1-180000] | Slow SQL threshold |
| LOAD_DATA_BATCH_INSERT_SIZE | No | 1024 | [1-180000] | The number of records for each batch insert of LOAD DATA |
| LOAD_DATA_CACHE_BUFFER_SIZE | No | 60 | [1-180000] | LOAD DATA cache size, default 60Mb, mainly for flow control |
| MAX_ALLOWED_PACKET | No | 16777216 | [4194304-33554432] | Maximum packet size |
| KILL_CLOSE_STREAM | No | FALSE | [FALSE/TRUE] | Whether to enable the physical connection streaming early stop function |
| ALLOW_SIMPLE_SEQUENCE | No | FALSE | [FALSE/TRUE] | Whether simple sequence is allowed |
| MAX_PARAMETERIZED_SQL_LOG_LENGTH | No | 5000 | [1-1000000] | The maximum length of parameterized SQL log printing |
| FORBID_EXECUTE_DML_ALL | No | TRUE | [TRUE/FALSE] | Whether to prohibit full table delete/update |
| GROUP_SEQ_CHECK_INTERVAL | Yes | 60 | [1-36000] | Period/interval for checking to insert explicit values, in seconds |
| JOIN_BLOCK_SIZE | Yes | 300 | [1-100000] | The number of IN Values ​​when BKAJOIN is executed under non-dynamic clipping |
| LOOKUP_JOIN_MAX_BATCH_SIZE | Yes | 6400 | [1-100000] | The maximum number of IN Values ​​when BKAJOIN is executed |
| LOOKUP_JOIN_MIN_BATCH_SIZE | Yes | 100 | [1-100000] | The maximum number of IN Values ​​when BKAJOIN is executed |
| PURGE_TRANS_INTERVAL | Yes | 300 | [1-180000] | Transaction log purge interval |
| PURGE_TRANS_BEFORE | Yes | 1800 | [1-180000] | How long ago to purge the transaction log |
| ENABLE_BACKGROUND<br />_STATISTIC_COLLECTION | No | TRUE | [TRUE/FALSE] | Whether statistical data collection is allowed |
| GENERAL_DYNAMIC_SPEED_LIMITATION | No | -1 | [-1-10000000] | Data backfilling, verification dynamic speed limit adjustment, -1 is the default limit |
| PARALLELISM | No | -1 | [1-1024] | Parallelism of a single machine, the default is derived from the specification |
| LOGICAL_DB_TIME_ZONE | No | SYSTEM | [SYSTEM/±HH:mm] | Database time zone |
| MPP_PARALLELISM | No | -1 | [1-1024] | Concurrency of MPP execution mode, the default is derived from the specification |
| DATABASE_PARALLELISM | No | 0 | [0-1024] | The number of SQLs that can be issued simultaneously on a DN for a single query, and the user calculates the concurrency of Scan |
| POLARDBX_PARALLELISM | No | 0 | [0-1024] | The maximum concurrency allowed by a single query in one CN, the default is the number of CPU cores |
| MPP_METRIC_LEVEL | No | 3 | [0/1/2/3] | The degree of statistical information collection during the calculation process, the higher the level, the finer the collection granularity |
| ENABLE_COMPLEX_DML_CROSS_DB | No | TRUE | [TRUE/FALSE] | Whether to support cross-database complex DML |
| PER_QUERY_MEMORY_LIMIT | Yes | -1 | [-1-9223372036854775807] | Query-level memory pool size limit, the default is one third of the global connection pool |
| ENABLE_SPILL | No | FALSE | [FALSE/TRUE] | Temporary table dump switch |
| CONN_POOL_MIN_POOL_SIZE | No | 20 | [0-10] | The minimum number of physical sub-library links |
| CONN_POOL_MAX_POOL_SIZE | No | 60 | [1-1600] | The maximum number of physical sub-library links |
| CONN_POOL_MAX_WAIT_THREAD_COUNT | No | 0 | [-1-8192] | The maximum number of connections waiting for a single sub-database (DRUID) |
| CONN_POOL_IDLE_TIMEOUT | No | 30 | [1-60] | Physical idle link timeout |
| CONN_POOL_BLOCK_TIMEOUT | No | 5000 | [1000-60000] | The maximum waiting time for the physical connection pool to obtain a connection |
| CONN_POOL_XPROTO_MAX<br />_POOLED_SESSION_PER_INST | No | 512 | [1-8192] | The maximum cache session number of a single storage node (private protocol) |
| XPROTO_MAX_DN_CONCURRENT | No | 500 | [1-8192] | Maximum number of concurrent requests for a single storage node (private protocol) |
| XPROTO_MAX_DN_WAIT_CONNECTION | No | 32 | [1-8192] | The maximum request waiting number of a single storage node (private protocol) |
| MERGE_SORT_BUFFER_SIZE | No | 2048 | [1024-81920] | The buffer size used by the TableScan layer for merge sorting, the default is 2Mb |
| WORKLOAD_TYPE | No | | [AP/TP] | Whether to specify the workload of the query, the default is to intelligently identify the load based on the cost |
| EXECUTOR_MODE | No | | [MPP/TP_LOCAL/AP_LOCAL] | Whether to specify the execution mode of the query, the default is to select the execution mode based on workload |
| ENABLE_MASTER_MPP | No | FALSE | [TRUE/FALSE] | Whether to enable the MPP capability on the master instance |
| LOOKUP_JOIN_BLOCK_SIZE_PER_SHARD | Yes | 50 | [1-100000] | The number of IN Values ​​agreed upon in a single shard when BKAJOIN is executed under pruning |
| ENABLE_RUNTIME_FILTER | NO | TRUE | [TRUE/FALSE] | Runtime Filter switch |
| FEEDBACK_WORKLOAD_AP_THRESHOLD | No | FALSE | [TRUE/FALSE] | HTAP FEEDBACK switch for AP query |
| FEEDBACK_WORKLOAD_TP_THRESHOLD | NO | FALSE | [TRUE/FALSE] | HTAP FEEDBACK switch for TP queries |
| MASTER_READ_WEIGHT | No | -1 | [0-100] | Rule-based read-write split weight |
| SHOW_ALL_PARAMS | No | FALSE | [TRUE/FALSE] | Show SHOW all variables |
| ENABLE_SET_GLOBAL | No | FALSE | [TRUE/FALSE] | Enable SET GLOBAL statement switch |
| FORCE_READ_OUTSIDE_TX | No | FALSE | [TRUE/FALSE] | Whether to forcibly open multiple connections on a sub-database within a transaction |
| ENABLE_COROUTINE | Yes | FALSE | [TRUE/FALSE] | Whether to enable the wisp coroutine |
| TRUNCATE_TABLE_WITH_GSI | No | FALSE | [TRUE/FALSE] | Whether to allow truncate tables containing gsi |
| DDL_ON_GSI | No | FALSE | [TRUE/FALSE] | Whether to allow DDL directly in the GSI table |
| DML_ON_GSI | No | FALSE | [TRUE/FALSE] | Whether to allow DML directly in the GSI table |
| ENABLE_HASH_JOIN | No | TRUE | [TRUE/FALSE] | Whether to allow generation of HashJoin nodes during query planning optimization |
| ENABLE_BKA_JOIN | No | TRUE | [TRUE/FALSE] | Whether to allow BKAJoin node generation during query plan optimization |
| ENABLE_NL_JOIN | No | TRUE | [TRUE/FALSE] | Whether to allow NLJoin node generation during query plan optimization |
| ENABLE_SEMI_NL_JOIN | No | TRUE | [TRUE/FALSE] | Whether it is allowed to convert SemiJoin to NLJoin during query plan optimization |
| ENABLE_SEMI_HASH_JOIN | No | TRUE | [TRUE/FALSE] | Whether it is allowed to convert SemiJoin into HashJoin during query plan optimization |
| ENABLE_SEMI_BKA_JOIN | No | TRUE | [TRUE/FALSE] | Whether it is allowed to convert SemiJoin into BKAJoin during query plan optimization |
| ENABLE_SEMI_SORT_MERGE_JOIN | No | TRUE | [TRUE/FALSE] | Whether it is allowed to convert SemiJoin into MergeJoin during query plan optimization |
| ENABLE_MATERIALIZED_SEMI_JOIN | No | TRUE | [TRUE/FALSE] | Whether it is allowed to convert SemiJoin into MaterializedJoin during query plan optimization |
| ENABLE_SEMI_JOIN_REORDER | No | TRUE | [TRUE/FALSE] | Whether SemiJoin is allowed to participate in CBO Reorder optimization during query plan optimization |
| ENABLE_HASH_AGG | No | TRUE | [TRUE/FALSE] | Whether to allow HashAgg node generation during query plan optimization |
| ENABLE_PARTIAL_AGG | No | TRUE | [TRUE/FALSE] | Whether Agg is allowed to be split into two phases during query plan optimization |
| ENABLE_SORT_AGG | No | TRUE | [TRUE/FALSE] | Whether to allow SortAgg node generation during query plan optimization |
| ENABLE_PUSH_PROJECT | No | TRUE | [TRUE/FALSE] | Whether to allow Project PushDown during query plan optimization |
| ENABLE_PUSH_JOIN | No | TRUE | [TRUE/FALSE] | Whether to allow Join PushDown during query plan optimization |
| ENABLE_PUSH_AGG | No | TRUE | [TRUE/FALSE] | Whether to allow Agg PushDown during query plan optimization |
| ENABLE_CBO_PUSH_AGG | No | TRUE | [TRUE/FALSE] | Whether to allow Agg to transparently transmit Join during query plan optimization |
| ENABLE_PUSH_SORT | No | TRUE | [TRUE/FALSE] | Whether to allow Sort PushDown during query plan optimization |
| ENABLE_STATISTIC_FEEDBACK | No | TRUE | [TRUE/FALSE] | Whether to support Feedback for statistical information correction |
| ENABLE_CBO_PUSH_JOIN | No | TRUE | [TRUE/FALSE] | Whether to allow Join transparent transmission optimization during query plan optimization |
| ENABLE_SORT_JOIN_TRANSPOSE | No | TRUE | [TRUE/FALSE] | Whether to allow Sort to transparently transmit Join during query plan optimization |
| CHUNK_SIZE | No | 1024 | [1-10240] | Set the batch size for each calculation of the executor |
| ENABLE_SORT_MERGE_JOIN | No | TRUE | [TRUE/FALSE] | Whether to disable MergeJoin node generation during query plan optimization |
| ENABLE_BKA_PRUNING | No | TRUE | [TRUE/FALSE] | Whether to enable the pruning function of BKAJoin |
| ENABLE_SPM | No | TRUE | [TRUE/FALSE] | Whether to enable execution plan management |
| ENABLE_EXPRESSION_VECTORIZATION | No | TRUE | [TRUE/FALSE] | Whether to enable expression vectorization |
| FORCE_DDL_ON_LEGACY_ENGINE | No | TRUE | [TRUE/FALSE] | Whether to enable the new DDL engine |
| PURE_ASYNC_DDL_MODE | No | TRUE | [TRUE/FALSE] | Whether to execute the ddl task in a non-blocking manner, default enabled means that the client returns immediately after executing the ddl, check the execution status through show [full] ddl |
| DDL_JOB_REQUEST_TIMEOUT | No | 90000 | [1-9223372036854775807] | Set the maximum timeout time for DDL execution, the default is 25 days |
| LOGICAL_DDL_PARALLELISM | No | 1 | [1-10240] | Configure the concurrency of logical DDL execution, setting it to 1 means to execute DDL tasks serially |
| ENABLE_BROADCAST_RANDOM_READ | No | TRUE | [TRUE/FALSE] | Whether to enable broadcast table random read optimization |



