storage layer variables
==========================

PolarDB-X is a storage-computing separation architecture. The variable names and meanings of the storage layer (DN) are aligned with those of MySQL. Here are the common DN variables (configurable in the console). For other variables, please refer to [MySQL Variables](https:// dev.mysql.com/doc/refman/5.7/en/server-system-variables.html).


| Name | Whether to restart | Default value | Modification range | Remarks |
| ---------------------------------------- | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| loose_enable_gts | yes | 1 | [0/1] | global timestamp switch |
| loose_gts_lease | yes | 5000 | [0-10000] | lease time for global timestamp |
| loose_performance-schema-instrument | No | wait/lock/metadata/sql/mdl=ON | .* | Used to obtain MDL lock information when MySQL is running |
| performance_schema | No | ON | [ON,OFF] | Used to monitor the execution details of the MySQL server |
| binlog_rows_query_key_content | No | ON | [ON,OFF] | Print sql to binlog log |
| loose_polarx_max_allowed_packet | No | 16777216 | (0-2147483648) | Limit the size of packets received and sent by Server under private protocols |
| innodb_buffer_pool_load_at_startup | Yes | ON | [ON/OFF] | Whether to reload the buffer pool after startup |
| bulk_insert_buffer_size | No | 4194304 | [0-4294967295] | Used to cache the number of temporary buffer writes when inserting data in batches |
| show_old_temporals                       | 否       | OFF                                                          | [ON/OFF]                                                     |                                                              |
| ft_query_expansion_limit | Yes | 20 | [0-1000] | Specify the maximum number of matches for full-text search of MyISAM engine tables using with query expansion |
| innodb_old_blocks_time | No | 1000 | [0-1024] | It is used to indicate how long the page needs to wait to be added to the hot end of the LRU list after the page is read to the mid position |
| innodb_stats_sample_pages | No | 8 | [1-4294967296] | Control collection precision |
| thread_stack | Yes | 262144 | [131072-18446744073709551615] | When each connection thread is created, the memory size allocated to it by MySQL |
| lc_time_names | No | en_US | [ja_JP/pt_BR/en_US] | Controls the language used to display day and month names and abbreviations |
| innodb_thread_concurrency | No | 0 | [0-1000] | Concurrency limit |
| default_time_zone | is | SYSTEM | [SYSTEM/-12:00/-11:00/-10:00/-9:00/-8:00/-7:00/<br />-6:00/- 5:00/-4:00/-3:00/-2:00/-1:00/+0:00/+1:00/<br />+2:00/+3:00/+4 :00/+5:00/+5:30/+5:45/+6:00/+6:30/<br />+7:00/+8:00/+9:00/+10: 00/+11:00/+12:00/+13:00] | Time Zone Settings |
| old_passwords | Negative | 0 | [0/2] |
| optimizer_search_depth | No | 62 | [0-62] | In multi-table association scenarios, control the recursion depth of the optimizer |
| innodb_compression_level | No | 6 | [0-9] |
| loose_innodb_log_optimize_ddl            | 否       | OFF                                                          | [ON/OFF]                                                     |                                                              |
| max_sort_length                          | 否       | 1024                                                         | [4-8388608]                                                  |                                                              |
| slave_pending_jobs_size_max              | 否       | 1073741824                                                   | [1024-18446744073709551615]                                  |                                                              |
| innodb_online_alter_log_max_size         | 否       | 134217728                                                    | [134217728-2147483647]                                       |                                                              |
| key_cache_block_size                     | 否       | 1024                                                         | [512-16384]                                                  |                                                              |
| mysql_native_password_proxy_users        | 否       | OFF                                                          | [ON/OFF]                                                     |                                                              |
| innodb_adaptive_max_sleep_delay          | 否       | 150000                                                       | [1-1000000]                                                  |                                                              |
| innodb_purge_rseg_truncate_frequency     | 否       | 128                                                          | [1-128]                                                      |                                                              |
| query_alloc_block_size                   | 否       | 8192                                                         | [1024-16384]                                                 |                                                              |
| innodb_lock_wait_timeout                 | 否       | 50                                                           | [1-1073741824]                                               |                                                              |
| innodb_purge_threads | yes | 1 | [1-32] | |
| innodb_compression_failure_threshold_pct | 否       | 5                                                            | [0-100]                                                      |                                                              |
| innodb_compression_pad_pct_max           | 否       | 50                                                           | [0-70]                                                       |                                                              |
| binlog_rows_query_log_events             | 否       | ON                                                           | [ON/OFF]                                                     |                                                              |
| innodb_stats_persistent_sample_pages     | 否       | 20                                                           | [0-4294967295]                                               |                                                              |
| innodb_ft_total_cache_size               | 是       | 640000000                                                    | [32000000-1600000000]                                        |                                                              |
| innodb_flush_method                      | 是       | O_DIRECT                                                     | [fsync/O_DSYNC/littlesync/<br />nosync/O_DIRECT/O_DIRECT_NO_FSYNC] |                                                              |
| eq_range_index_dive_limit                | 否       | 10                                                           | [0-4294967295]                                               |                                                              |
| loose_max_execution_time                 | 否       | 0                                                            | [0-4294967295]                                               |                                                              |
| loose_optimizer_trace_features           | 否       | greedy_search=on,range_optimizer=on,<br />dynamic_range=on,repeated_subselect=on | .*                                                           |                                                              |
| rds_reserved_connections                 | 否       | 512                                                          | [0-512]                                                      |                                                              |
| connect_timeout | no | 10 | [1-3600] |
| innodb_purge_batch_size | yes | 300 | [1-5000] | |
| div_precision_increment                  | 否       | 4                                                            | [0-30]                                                       |                                                              |
| avoid_temporal_upgrade                   | 否       | OFF                                                          | [ON/OFF]                                                     |                                                              |
| innodb_sync_array_size                   | 是       | 1                                                            | [1-64]                                                       |                                                              |
| sync_binlog | No | 1 | [0-2147483647] |
| innodb_stats_method                      | 否       | nulls_equal                                                  | [nulls_equal/nulls_unequal/nulls_ignored]                    |                                                              |
| lock_wait_timeout                        | 否       | 31536000                                                     | [1-1073741824]                                               |                                                              |
| net_read_timeout | no | 30 | [1-18446744073709551615] |
| innodb_deadlock_detect                   | 否       | ON                                                           | [ON/OFF]                                                     |                                                              |
| innodb_write_io_threads                  | 是       | 4                                                            | [1-64]                                                       |                                                              |
| end_markers_in_json | No | OFF | [ON/OFF] |
| ngram_token_size |是 | 2| [0-20] | |
| loose_innodb_numa_interleave             | 是       | ON                                                           | [ON/OFF]                                                     |                                                              |
| max_binlog_stmt_cache_size | No | 18446744073709547520 | [4096-18446744073709547520] |
| innodb_checksum_algorithm                | 否       | crc32                                                        | [innodb/crc32/none/strict_innodb/strict_crc32/strict_none]   |                                                              |
| query_cache_type | yes | 0 | [0/1/2] | |
| innodb_ft_enable_diag_print              | 否       | OFF                                                          | [ON/OFF]                                                     |                                                              |
| innodb_ft_enable_stopword                | 否       | ON                                                           | [ON/OFF]                                                     |                                                              |
| innodb_io_capacity | No | 20000 | [0-18446744073709551615] |
| slow_launch_time | no | 2 | [1-1024] |
| innodb_table_locks                       | 否       | ON                                                           | [ON/OFF]                                                     |                                                              |
| innodb_stats_persistent                  | 否       | ON                                                           | [ON/OFF]                                                     |                                                              |
| tmp_table_size                           | 否       | 2097152                                                      | [262144-134217728]                                           |                                                              |
| disconnect_on_expired_password           | 是       | ON                                                           | [ON/OFF]                                                     |                                                              |
| default_storage_engine                   | 是       | InnoDB                                                       | [InnoDB/innodb]                                              |                                                              |
| net_retry_count | No | 10 | [1-4294967295] |
| innodb_ft_cache_size | yes | 8000000 | [1600000-80000000] | |
| binlog_cache_size | No | 2097152 | [4096-16777216] |
| innodb_max_dirty_pages_pct               | 否       | 75                                                           | [0-99]                                                       |                                                              |
| query_cache_limit | No | 1048576 | [1-1048576] | |
| innodb_disable_sort_file_cache           | 否       | ON                                                           | [ON/OFF]                                                     |                                                              |
| innodb_lru_scan_depth                    | 否       | 1024                                                         | [100-18446744073709551615]                                   |                                                              |
| innodb_ft_result_cache_limit             | 否       | 2000000000                                                   | [1000000-4294967295]                                         |                                                              |
| long_query_time | no | 1 | [0.03-31536000] |
interactive_timeout | no | 7200 | [10-86400] | |
| innodb_read_io_threads                   | 是       | 4                                                            | [1-64]                                                       |                                                              |
| transaction_prealloc_size                | 否       | 4096                                                         | [1024-131072]                                                |                                                              |
| open_files_limit | yes | 65535 | [1-18446744073709551615] | |
| innodb_open_files | yes | 3000 | [10-4294967295] | |
| max_heap_table_size                      | 否       | 67108864                                                     | [16384-1844674407370954752]                                  |                                                              |
| automatic_sp_privileges | No | ON | [ON/OFF] |
| explicit_defaults_for_timestamp          | 否       | OFF                                                          | [ON/OFF]                                                     |                                                              |
| ft_max_word_len | yes | 84 | [10-4294967295] | |
| innodb_autoextend_increment              | 否       | 64                                                           | [1-1000]                                                     |                                                              |
| sql_mode                                 | 否       |                                                              | (\s\*/REAL_AS_FLOAT/PIPES_AS_CONCAT/<br />ANSI_QUOTES/IGNORE_SPACE/<br />ONLY_FULL_GROUP_BY/<br />NO_UNSIGNED_SUBTRACTION/<br />NO_DIR_IN_CREATE/POSTGRESQL/<br />ORACLE/MSSQL/DB2/MAXDB/<br />NO_KEY_OPTIONS/NO_TABLE_OPTIONS/<br />NO_FIELD_OPTIONS/MYSQL323/MYSQL40/<br />ANSI/NO_AUTO_VALUE_ON_ZERO/<br />NO_BACKSLASH_ESCAPES/<br />STRICT_TRANS_TABLES/STRICT_ALL_TABLES/<br />NO_ZERO_IN_DATE/NO_ZERO_DATE/<br />ALLOW_INVALID_DATES/<br />ERROR_FOR_DIVISION_BY_ZERO/<br />TRADITIONAL/HIGH_NOT_PRECEDENCE/<br />NO_ENGINE_SUBSTITUTION/<br />PAD_CHAR_TO_FULL_LENGTH/<br />NO_AUTO_CREATE_USER)(,NO_AUTO_CREATE_USER/<br />,REAL_AS_FLOAT/,PIPES_AS_CONCAT/<br />,ANSI_QUOTES/,IGNORE_SPACE/<br />,ONLY_FULL_GROUP_BY/<br />,NO_UNSIGNED_SUBTRACTION/,NO_DIR_IN_CREATE/<br />,POSTGRESQL/,ORACLE/,MSSQL/,DB2/,MAXDB/<br />,NO_KEY_OPTIONS/,NO_TABLE_OPTIONS/<br />,NO_FIELD_OPTIONS/,MYSQL323/,MYSQL40/,ANSI/<br />,NO_AUTO_VALUE_ON_ZERO/,NO_BACKSLASH_ESCAPES/<br />,STRICT_TRANS_TABLES/,STRICT_ALL_TABLES/<br />,NO_ZERO_IN_DATE/,NO_ZERO_DATE/<br />,ALLOW_INVALID_DATES/,ERROR_FOR_DIVISION_BY_ZERO/<br />,TRADITIONAL/,HIGH_NOT_PRECEDENCE/<br />,NO_ENGINE_SUBSTITUTION/,PAD_CHAR_TO_FULL_LENGTH)* |                                                              |
| innodb_stats_transient_sample_pages      | 否       | 8                                                            | [1-4294967295]                                               |                                                              |
| innodb_random_read_ahead                 | 否       | OFF                                                          | [ON/OFF]                                                     |                                                              |
| session_track_state_change               | 否       | OFF                                                          | [ON/OFF]                                                     |                                                              |
| table_open_cache | No | 2000 | [1-524288] | Size of table file handle cache |
| range_optimizer_max_mem_size             | 否       | 8388608                                                      | [0-18446744073709551615]                                     |                                                              |
| innodb_status_output                     | 否       | OFF                                                          | [ON/OFF]                                                     |                                                              |
| innodb_log_compressed_pages              | 否       | OFF                                                          | [ON/OFF]                                                     |                                                              |
| slave_net_timeout | No | 60 | [15-300] |
| delay_key_write                          | 否       | ON                                                           | [ON/OFF/ALL]                                                 |                                                              |
| query_cache_wlock_invalidate             | 否       | OFF                                                          | [ON/OFF]                                                     |                                                              |
| max_points_in_geometry                   | 否       | 65536                                                        | [3-1048576]                                                  |                                                              |
| max_prepared_stmt_count | no | 16382 | [0-1048576] |
wait_timeout | no | 86400 | [1-31536000] | |
| query_cache_min_res_unit | No | 1024 | [512-18446744073709551608] | Result cache configuration |
| innodb_print_all_deadlocks               | 否       | OFF                                                          | [OFF/ON]                                                     |                                                              |
| loose_thread_pool_size | No | 32 | [1-1024] | Number of pools, default: 32. The threads in the thread pool are evenly divided into multiple groups for management |
| binlog_stmt_cache_size | No | 32768 | [4096-16777216] |
| transaction_isolation | No | READ-COMMITTED | [READ-UNCOMMITTED/READ-COMMITTED/REPEATABLE-READ/SERIALIZABLE] | Transaction isolation level policy |
| innodb_buffer_pool_dump_at_shutdown      | 否       | ON                                                           | [ON/OFF]                                                     |                                                              |
| query_prealloc_size                      | 否       | 8192                                                         | [8192-1048576]                                               |                                                              |
| key_cache_age_threshold                  | 否       | 300                                                          | [100-4294967295]                                             |                                                              |
| loose_rds_kill_connections               | 否       | 20                                                           | [0, 18446744073709551615]                                    |                                                              |
| transaction_alloc_block_size             | 否       | 8192                                                         | [1024-131072]                                                |                                                              |
| optimizer_trace_limit                    | 否       | 1                                                            | [0-4294967295]                                               |                                                              |
| metadata_locks_cache_size                | 是       | 1024                                                         | [1-1048576]                                                  |                                                              |
| optimizer_prune_level                    | 否       | 1                                                            | [0/1]                                                        |                                                              |
| innodb_max_purge_lag | no | 0 | [0-4294967295] |
| innodb_buffer_pool_dump_pct              | 否       | 25                                                           | [1-100]                                                      |                                                              |
| innodb_max_dirty_pages_pct_lwm |否 | 0 | [0-99] | |
| max_sp_recursion_depth | no | 0 | [0-255] |
| innodb_status_output_locks               | 否       | OFF                                                          | [ON/OFF]                                                     |                                                              |
| updatable_views_with_limit               | 否       | YES                                                          | [YES/NO]                                                     |                                                              |
| binlog_row_image                         | 否       | full                                                         | [full/minimal]                                               |                                                              |
| innodb_change_buffer_max_size            | 否       | 25                                                           | [0-50]                                                       |                                                              |
| innodb_optimize_fulltext_only            | 否       | OFF                                                          | [ON/OFF]                                                     |                                                              |
| loose_opt_rds_last_error_gtid            | 否       | ON                                                           | [ON/OFF]                                                     |                                                              |
| innodb_ft_max_token_size                 | 是       | 84                                                           | [10-84]                                                      |                                                              |
| innodb_max_undo_log_size                 | 否       | 1073741824                                                   | [10485760-18446744073709551615]                              |                                                              |
| slave_parallel_type                      | 否       | LOGICAL_CLOCK                                                | DATABASE,LOGICAL_CLOCK                                       |                                                              |
| loose_rds_check_core_file_enabled        | 否       | ON                                                           | [ON/OFF]                                                     |                                                              |
| innodb_adaptive_hash_index               | 否       | ON                                                           | [ON/OFF]                                                     |                                                              |
| innodb_sync_spin_loops                   | 否       | 30                                                           | [0-4294967295]                                               |                                                              |
| net_write_timeout                        | 否       | 60                                                           | [1-18446744073709551615]                                     |                                                              |
flush_time | no | 0 | [0-31536000] | |
| lower_case_table_names                   | 是       | 1                                                            | [0/1]                                                        |                                                              |
| sha256_password_proxy_users              | 否       | OFF                                                          | [ON/OFF]                                                     |                                                              |
| character_set_filesystem                 | 否       | binary                                                       | [utf8/latin1/gbk/binary]                                     |                                                              |
| innodb_flush_sync                        | 否       | ON                                                           | [ON/OFF]                                                     |                                                              |
| tls_version                              | 是       | TLSv1,TLSv1.1,TLSv1.2                                        | [TLSv1,TLSv1.1,TLSv1.2/TLSv1,TLSv1.1]                        |                                                              |
| key_cache_division_limit                 | 否       | 100                                                          | [1-100]                                                      |                                                              |
| delayed_insert_timeout                   | 否       | 300                                                          | [1-3600]                                                     |                                                              |
| preload_buffer_size                      | 否       | 32768                                                        | [1024-1073741824]                                            |                                                              |
| innodb_read_ahead_threshold              | 否       | 56                                                           | [0-1024]                                                     |                                                              |
| loose_optimizer_switch                   | 否       | index_merge=on,index_merge_union=on,<br />index_merge_sort_union=on,<br />index_merge_intersection=on,<br />engine_condition_pushdown=on,<br />index_condition_pushdown=on,<br />mrr=on,mrr_cost_based=on,<br />block_nested_loop=on,<br />batched_key_access=off,<br />materialization=on,semijoin=on,<br />loosescan=on,firstmatch=on,<br />subquery_materialization_cost_based=on,<br />use_index_extensions=on | .*                                                           |                                                              |
| concurrent_insert | No | 1 | [0/1/2] | Concurrent insert function setting |
| block_encryption_mode                    | 否       | "aes-128-ecb"                                                | ["aes-128-ecb"/"aes-192-ecb"/"aes-256-ecb"/<br />"aes-128-cbc"/"aes-192-cbc"/"aes-256-cbc"] |                                                              |
| slow_query_log | No | ON | [ON/OFF] | Record slow log |
| net_buffer_length                        | 否       | 16384                                                        | [1024-1048576]                                               |                                                              |
| query_cache_size                         | 否       | 3145728                                                      | [0-104857600]                                                |                                                              |
| delayed_insert_limit                     | 否       | 100                                                          | [1-4294967295]                                               |                                                              |
| innodb_large_prefix                      | 否       | OFF                                                          | [ON/OFF]                                                     |                                                              |
| innodb_monitor_disable                   | 否       |                                                              | all                                                          |                                                              |
| innodb_adaptive_flushing_lwm             | 否       | 10                                                           | [0-70]                                                       |                                                              |
| innodb_log_checksums                     | 否       | ON                                                           | [ON/OFF]                                                     |                                                              |
| delayed_queue_size                       | 否       | 1000                                                         | [1-4294967295]                                               |                                                              |
| session_track_gtids                      | 否       | OFF                                                          | [OFF/OWN_GTID/ALL_GTIDS]                                     |                                                              |
| innodb_thread_sleep_delay                | 否       | 10000                                                        | [0-1000000]                                                  |                                                              |
| loose_rds_set_connection_id_enabled      | 否       | ON                                                           | [ON/OFF]                                                     |                                                              |
| innodb_old_blocks_pct                    | 否       | 37                                                           | [5-95]                                                       |                                                              |
| innodb_ft_sort_pll_degree                | 是       | 2                                                            | [1-16]                                                       |                                                              |
| log_slow_admin_statements                | 否       | OFF                                                          | [ON/OFF]                                                     |                                                              |
| innodb_stats_on_metadata                 | 否       | OFF                                                          | [ON/OFF]                                                     |                                                              |
| stored_program_cache | no | 256 | [16-524288] |
group_concat_max_len | no | 1024 | [4-1844674407370954752] | |
| innodb_sort_buffer_size                  | 是       | 1048576                                                      | [65536-67108864]                                             |                                                              |
| innodb_page_cleaners                     | 是       | 1                                                            | [1-64]                                                       |                                                              |
| innodb_spin_wait_delay                   | 否       | 6                                                            | [0-4294967295]                                               |                                                              |
| myisam_sort_buffer_size                  | 否       | 262144                                                       | [262144-16777216]                                            |                                                              |
| innodb_rollback_segments                 | 否       | 128                                                          | [1-128]                                                      |                                                              |
| innodb_commit_concurrency                | 是       | 0                                                            | [0-1000]                                                     |                                                              |
| innodb_concurrency_tickets               | 否       | 5000                                                         | [1-4294967295]                                               |                                                              |
| table_definition_cache                   | 否       | 512                                                          | [400-524288]                                                 |                                                              |
| auto_increment_increment                 | 否       | 1                                                            | [1-65535]                                                    |                                                              |
| binlog_checksum | is | CRC32 | | |
| max_seeks_for_key                        | 否       | 18446744073709500000                                         | [1-18446744073709551615]                                     |                                                              |
| sync_relay_log                           | 否       | 1                                                            | [0-2147483647]                                               |                                                              |
| max_length_for_sort_data                 | 否       | 1024                                                         | [0-838860]                                                   |                                                              |
| back_log | yes | 3000 | [0-65535] | |
| max_error_count | no | 64 | [0-65535] |
| innodb_io_capacity_max | no | 40000 | [0-18446744073709551615] |
| innodb_strict_mode | No | OFF | [ON/OFF] |
| binlog_order_commits | yes | OFF | | |
| min_examined_row_limit | No | 0 | [0-4294967295] |
| innodb_ft_min_token_size | yes | 3 | [0-16] | |
| innodb_stats_auto_recalc                 | 否       | ON                                                           | [ON/OFF]                                                     |                                                              |
| max_connect_errors                       | 否       | 100                                                          | [0-4294967295]                                               |                                                              |
| session_track_schema                     | 否       | ON                                                           | [ON/OFF]                                                     |                                                              |
| join_buffer_size                         | 否       | 262144                                                       | [128-4294967295]                                             |                                                              |
| innodb_change_buffering                  | 否       | all                                                          | [none/inserts/deletes/changes/purges/all]                    |                                                              |
| optimizer_trace_max_mem_size             | 否       | 16384                                                        | [0-4294967295]                                               |                                                              |
| innodb_autoinc_lock_mode                 | 是       | 2                                                            | [0/1/2]                                                      |                                                              |
| innodb_rollback_on_timeout               | 是       | OFF                                                          | [OFF/ON]                                                     |                                                              |
| loose_opt_rds_enable_show_slave_lag      | 否       | ON                                                           | [ON/OFF]                                                     |                                                              |
| max_write_lock_count                     | 否       | 102400                                                       | [1-102400]                                                   |                                                              |
| master_verify_checksum | yes | OFF | | |
| innodb_ft_num_word_optimize              | 否       | 2000                                                         | [0-10000]                                                    |                                                              |
| max_join_size | no | 18446744073709551615 | [1-18446744073709551615] |
| loose_validate_password_length           | 否       | 8                                                            | [1-12]                                                       |                                                              |
| log_throttle_queries_not_using_indexes   | 否       | 0                                                            | [0-4294967295]                                               |                                                              |
| innodb_max_purge_lag_delay               | 否       | 0                                                            | [0-10000000]                                                 |                                                              |
| loose_optimizer_trace                    | 否       | enabled=off,one_line=off                                     | .*                                                           |                                                              |
| loose_thread_handling                    | 是       | one-thread-per-connection                                    | [one-thread-per-connection/pool-of-threads]                  |                                                              |
| default_week_format                      | 否       | 0                                                            | [0-7]                                                        |                                                              |
| innodb_cmp_per_index_enabled             | 否       | OFF                                                          | [ON/OFF]                                                     |                                                              |
| show_compatibility_56                    | 否       | OFF                                                          | [ON/OFF]                                                     |                                                              |
| host_cache_size | No | 644 | [0-65535] | host cache size |
| low_priority_updates                     | 否       | 0                                                            | [0/1]                                                        |                                                              |
| auto_increment_offset | No | 1 | [1-65535] | The auto-increment step of the auto-increment column |
| range_alloc_block_size                   | 否       | 4096                                                         | [4096-18446744073709551615]                                  |                                                              |
| ft_min_word_len | yes | 4 | [1-3600] | |
| sort_buffer_size | No | 262144 | [32768-4294967295] | Buffer for sorting applications |
| max_allowed_packet | No | 1073741824 | [16384-1073741824] | Limit the packet size accepted by the Server |
| thread_cache_size | No | 256 | [0-16384] | Execution thread cache |
| optimizer_trace_offset | no | -1 | [0-4294967295] |
| character_set_server | is | utf8 | [utf8/latin1/gbk/gb18030/utf8mb4] | character set at database level |
| innodb_adaptive_flushing | No | ON | [ON/OFF] | Adaptive flush dirty page switch |
| log_queries_not_using_indexes | No | OFF | [ON/OFF] | Queries that do not use indexes will not be recorded in the slow log |
| innodb_monitor_enable                    | 否       |                                                              | all                                                          |                                                              |
| table_open_cache_instances | Yes | 16 | [1-64] | Control the number of table cache instances |
| innodb_flush_neighbors | No | 1 | [0/1/2] | Used to control whether to flush other dirty pages adjacent to the dirty page to the disk when the buffer pool flushes dirty pages |
| innodb_buffer_pool_instances | yes | 1 | [1-64] | size of buffer pool instances |
| innodb_data_file_purge | No | OFF | [ON/OFF] | Whether to enable asynchronous purge policy. |
| innodb_data_file_purge_all_at_shutdown | No | OFF | [ON/OFF] | Purge all at shutdown. |
| innodb_data_file_purge_immediate | No | OFF | [ON/OFF] | Unlink data files without purge. |
| innodb_data_file_purge_interval | No | 100 | [1-1073741824] | Purge interval. Unit: ms. |
| innodb_data_file_purge_max_size | No | 512 | [1-1073741824] | The maximum size of a single file purge each time. Unit: MB. |
| hotspot | No | OFF | [ON/OFF] | Hotspot update switch |
| hotspot_lock_type | No | OFF | [ON/OFF] | Hotspot update lock type |



