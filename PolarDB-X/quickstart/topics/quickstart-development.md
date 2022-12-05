# Install and deploy by compiling source code

Ordinary developers usually use operating systems such as CentOS for development. This document explains how to participate in the process of PolarDB-X development from scratch, covering code compilation, database installation, deployment and other processes.

> **Remarks**: This document is mainly for CentOS7 and Ubuntu20 operating systems, and the principles of other Linux distributions are similar.

## Preparation

- Download [GalaxyEngine](https://github.com/ApsaraDB/galaxyengine) code, main branch
- Download [GalaxySQL](https://github.com/ApsaraDB/galaxysql) code, main branch
- Download [GalaxyGlue](https://github.com/ApsaraDB/galaxyglue) code, main branch
- Download [GalaxyCDC](https://github.com/ApsaraDB/galaxycdc) code, main branch

## Compile PolarDB-X DN (storage node, code name GalaxyEngine)

This step compiles and installs GalaxyEngine (mysql)

**Installation dependencies (CentOS7)**

```bash
yum install cmake3
ln -s /usr/bin/cmake3 /usr/bin/cmake


# install GCC7
yum install centos-release-scl
yum install devtoolset-7-gcc devtoolset-7-gcc-c++ devtoolset-7-binutils
echo "source /opt/rh/devtoolset-7/enable" >>/etc/profile

# install dependencies
yum install make automake git openssl-devel ncurses-devel bison libaio-devel
```

**Installation dependencies (Ubuntu20)**

```bash
# install GCC7
apt install -y gcc-7 g++-7
update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 60 \
--slave /usr/bin/g++ g++ /usr/bin/g++-7
update-alternatives --config gcc
gcc --version
g++ --version

# install dependencies
apt install make automake cmake git bison libaio-dev libncurses-dev libsasl2-dev libldap2-dev libssl-dev pkg-config
```

**compile**

```bash
# Enter the galaxyengine code path
cd galaxyengine

# Install boost1.70 (note: put boost in the warehouse to avoid downloading)
wget https://boostorg.jfrog.io/artifactory/main/release/1.70.0/source/boost_1_70_0.tar.gz
mkdir extra/boost
cp boost_1_70_0.tar.gz extra/boost/

# compile and install
# For detailed parameters, please refer to https://dev.mysql.com/doc/refman/8.0/en/source-configuration-options.html
cmake .                                 \
-DFORCE_INSOURCE_BUILD=ON           \
-DCMAKE_BUILD_TYPE="Debug"          \
-DSYSCONFDIR="/u01/mysql"           \
-DCMAKE_INSTALL_PREFIX="/u01/mysql" \
-DMYSQL_DATADIR="/u01/mysql/data"   \
-DMYSQL_MAINTAINER_MODE=0           \
-DWITH_SSL=openssl                  \
-DWITH_BOOST="./extra/boost/boost_1_70_0.tar.gz"
make -j8
make install
```


## Compile PolarDB-X CN (compute node, code name GalaxySQL)
This step compiles and installs the galaxysql & galaxyglue code.
```yaml
# Installation depends on JDK 1.8 and Maven 3

# Enter the code directory
cd galaxysql/

# Make sure the polardbx-rpc submodule (GalaxyGlue) is initialized
git submodule update --init

# Compile and package
mvn install -D maven.test.skip=true -D env=release

# Unzip and run
tar zxvf target/polardbx-server-5.4.12-SNAPSHOT.tar.gz
```

## Compile PolarDB-X CDC (log node, code name GalaxyCDC)
This step compiles and installs the galaxycdc code.
```yaml
# Enter the CDC code

# Compile and package
mvn install -D maven.test.skip=true -D env=release

# package in /polardbx-cdc-assemble/target/

# Unzip and run
tar zxvf polardbx-binlog.tar.gz
```

## Start PolarDB-X DN

- This step starts a mysql process as metadb and dn
- Refer to the mysql configuration file (my.cnf) in the appendix, and modify accordingly. By default, 4886 is used as the mysql port, and 32886 is used as the private protocol port
- By default, /u01/my3306 is used as the mysql data directory, which can be modified to other directories

> Note: Starting DN needs to be done with a non-root account

Start mysql:
```yaml
mkdir -p /u01/my3306/{data,log,run,tmp,mysql}
/u01/mysql/bin/mysqld --defaults-file=my.cnf --initialize-insecure
/u01/mysql/bin/mysqld --defaults-file=my.cnf
```


## Start PolarDB-X CN
After starting the mysql process, PolarDB-X can be initialized, and the following configurations need to be prepared:

- metadb user: The following uses `my_polarx`
- metadb database: Create a metadb library, the following uses `polardbx_meta_db_polardbx`
- Password encryption key (dnPasswordKey): `asdf1234ghjk5678` is used below
- PolarDB-X default username: defaults to `polarx_root`
- PolarDB-X default user password: the default is `123456`, which can be modified by the `-S` parameter

> Note: Starting CN needs to be done with a non-root account

Modify the configuration file conf/server.properties, and replace the following configuration items one by one:
```basic
# PolarDB-X service port
serverPort=8527
# PolarDB-X RPC port
rpcPort=9090
# MetaDB address
metaDbAddr=127.0.0.1:4886
# MetaDB private protocol port
metaDbXprotoPort=32886
# MetaDB user
metaDbUser=my_polarx
metaDbName=polardbx_meta_db_polardbx
# PolarDB-X instance name
instanceId=polardbx-polardbx
```


Initialize PolarDB-X:

- -I: Enter initialization mode
- -P: previously prepared dnPasswordKey
- -d: DataNode address list, in stand-alone mode is the port and address of the previously started mysql process
- -r: password to connect to metadb
- -u: root user created for PolarDB-X
- -S: root user password created for PolarDB-X

```sql
bin/startup.sh \
-I \
-P asdf1234ghjk5678 \
-d 127.0.0.1:4886:32886 \
-r "" \
-u polardbx_root \
-S "123456"
```

In this step, an internal password and an encrypted password will be generated, which need to be filled in the configuration file conf/server.properties for subsequent access:
```sql
Generate password for user: my_polarx && M8%V5%K9^$5%oY0%yC0+&1!J7@8+R6)
Encrypted password: DB84u4UkU/OYlMzu3aj9NFdknvxYgedFiW9z59bVnoc=
Root user for polarx with password: polardbx_root && 123456
Encrypted password for polarx: H1AzXc2NmCs61dNjH5nMvA==
======== Paste following configurations to conf/server.properties ! =======
black oscillator =
```


The last step, start PolarDB-X:
```yaml
bin/startup.sh -P asdf1234ghjk5678
```


Connect to PolarDB-X for verification. If it can be connected, it means that the database has started successfully, and you can happily run various SQL:
```sql
mysql -h127.1 -P8527 -upolardbx_root
```


## Start PolarDB-X CDC
After the PolarDB-X process is started, the PolarDB-X CDC component can be initialized, and the following configurations need to be prepared:

- metadb user: It is consistent with the value set when starting PolarDB-X, the following uses `my_polarx`
- metadb database: Consistent with the value set when starting PolarDB-X, the following uses `polardbx_meta_db_polardbx`
- metadb password: It is consistent with the value set when PolarDB-X is started, and ciphertext is required. The following uses `HMqvkvXZtT7XedA6t2IWY8+D7fJWIJir/mIY1Nf1b58=`
- metadb port: It is consistent with the value set when starting MySQL, the following uses `4886`
- Password encryption key (dnPasswordKey): consistent with the value set when starting PolarDB-X, the following uses `asdf1234ghjk5678`
- PolarDB-X username: It is consistent with the value set when starting PolarDB-X, and the default value `polardbx_root` is used below
- PolarDB-X user password: It is consistent with the value set when starting PolarDB-X, and ciphertext is required. The default value `H1AzXc2NmCs61dNjH5nMvA==` is used below
- PolarDB-X port: It is consistent with the value set when starting PolarDB-X, and the default value `8527` is used below
- The memory size allocated to CDC by the current machine: 16000 is used below, and the unit is M. Please replace the actual configuration value with the real value

> Note: Starting CDC needs to be done with a non-root account

Modify the configuration file conf/config.properties, and replace ${HOME} in the following example with the root directory of the current user, such as /home/mysql
```shell
useEncryptedPassword=true
polardbx.instance.id=polardbx-polardbx
mem_size=16000
metaDb_url=jdbc:mysql://127.0.0.1:4886/polardbx_meta_db_polardbx?useSSL=false
metaDb_username=my_polarx
black oscillator =
polarx_url=jdbc:mysql://127.0.0.1:8527/__cdc__
polarx_username=polardbx_root
polarx_password=H1AzXc2NmCs61dNjH5nMvA==
dnPasswordKey=asdf1234ghjk5678
storage.persistBasePath=${HOME}/logs/rocksdb
binlog.dir.path=${HOME}/binlog/
```
Next, you can start the CDC daemon process, the command is as follows. After startup, use the jps command to view the process status. CDC will have 3 subsidiary processes, namely DaemonBootStrap, TaskBootStrap and DumperBootStrap. The system log of CDC will be output to the ${HOME}/logs directory, and the global binlog log will be output to binlog. In the directory configured by the dir.path parameter, after the TaskBootStrap process and DumperBootStrap process are killed, they will be automatically pulled up by the Daemon process.
```shell
bin/daemon.sh start
```
Log in to PolarDB-X, perform some DDL or DML operations, and then execute the show binary logs and show binlog events commands to verify the generation status of the global binlog, enjoy it!

## Appendix
### mysql configuration file
```yaml
[mysqld]
socket = /u01/my3306/run/mysql.sock
datadir = /u01/my3306/data
tmpdir = /u01/my3306/tmp
log-bin = /u01/my3306/mysql/mysql-bin.log
log-bin-index = /u01/my3306/mysql/mysql-bin.index
# log-error = /u01/my3306/mysql/master-error.log
relay-log = /u01/my3306/mysql/slave-relay.log
relay-log-info-file = /u01/my3306/mysql/slave-relay-log.info
relay-log-index = /u01/my3306/mysql/slave-relay-log.index
master-info-file = /u01/my3306/mysql/master.info
slow_query_log_file = /u01/my3306/mysql/slow_query.log
innodb_data_home_dir = /u01/my3306/mysql
innodb_log_group_home_dir = /u01/my3306/mysql

port = 4886
loose_polarx_port = 32886
loose_galaxyx_port = 32886
loose_polarx_max_connections = 5000

loose_server_id = 476984231
loose_cluster-info = 127.0.0.1:14886@1
loose_cluster-id = 5431
loose_enable_gts = 1
loose_innodb_undo_retention=1800



core-file
loose_log_sql_info=1
loose_log_sql_info_index=1
loose_indexstat=1
loose_tablestat=1
default_authentication_plugin=mysql_native_password

# close 5.6 variables for 5.5
binlog_checksum=CRC32
log_bin_use_v1_row_events=on
explicit_defaults_for_timestamp=OFF
binlog_row_image=FULL
binlog_rows_query_log_events=ON
binlog_stmt_cache_size=32768

#innodb
innodb_data_file_path=ibdata1:100M;ibdata2:200M:autoextend
innodb_buffer_pool_instances=8
innodb_log_files_in_group=4
innodb_log_file_size=200M
innodb_log_buffer_size=200M
innodb_flush_log_at_trx_commit=1
#innodb_additional_mem_pool_size=20M #deprecated in 5.6
innodb_max_dirty_pages_pct=60
innodb_io_capacity_max=10000
innodb_io_capacity=6000
innodb_thread_concurrency=64
innodb_read_io_threads=8
innodb_write_io_threads=8
innodb_open_files=615350
innodb_file_per_table=1
innodb_flush_method=O_DIRECT
innodb_change_buffering=none
innodb_adaptive_flushing=1
#innodb_adaptive_flushing_method=keep_average #percona
#innodb_adaptive_hash_index_partitions=1      #percona
#innodb_fast_checksum=1                       #percona
#innodb_lazy_drop_table=0                     #percona
innodb_old_blocks_time=1000
innodb_stats_on_metadata=0
innodb_use_native_aio=1
innodb_lock_wait_timeout=50
innodb_rollback_on_timeout=0
innodb_purge_threads=1
innodb_strict_mode=1
#transaction-isolation=READ-COMMITTED
innodb_disable_sort_file_cache=ON
innodb_lru_scan_depth=2048
innodb_flush_neighbors=0
innodb_sync_array_size=16
innodb_print_all_deadlocks
innodb_checksum_algorithm=CRC32
innodb_max_dirty_pages_pct_lwm=10
innodb_buffer_pool_size=500M

#myisam
concurrent_insert=2
delayed_insert_timeout=300

#replication
slave_type_conversions="ALL_NON_LOSSY"
slave_net_timeout=4
skip-slave-start=OFF
sync_master_info=10000
sync_relay_log_info=1
master_info_repository=TABLE
relay_log_info_repository=TABLE
relay_log_recovery=0
slave_exec_mode=STRICT
#slave_parallel_type=DATABASE
slave_parallel_type=LOGICAL_CLOCK
loose_slave_pr_mode=TABLE
slave-parallel-workers=32

#binlog
server_id=193317851
binlog_cache_size=32K
max_binlog_cache_size=2147483648
loose_consensus_large_trx=ON
max_binlog_size=500M
max_relay_log_size=500M
relay_log_purge=OFF
binlog-format=ROW
sync_binlog=1
sync_relay_log=1
log-slave-updates=0
expire_logs_days=0
rpl_stop_slave_timeout=300
slave_checkpoint_group=1024
slave_checkpoint_period=300
slave_pending_jobs_size_max=1073741824
slave_rows_search_algorithms='TABLE_SCAN,INDEX_SCAN'
slave_sql_verify_checksum=OFF
master_verify_checksum=OFF

# parallel replay
binlog_transaction_dependency_tracking = WRITESET
transaction_write_set_extraction = XXHASH64


#got
gtid_mode=OFF
enforce_gtid_consistency=OFF

loose_consensus-io-thread_cnt=8
loose_consensus-worker-thread_cnt=8
loose_consensus_max_delay_index=10000
loose_consensus-election-timeout=10000
loose_consensus_max_packet_size=131072
loose_consensus_max_log_size=20M
loose_consensus_auto_leader_transfer=ON
loose_consensus_log_cache_size=536870912
loose_consensus_prefetch_cache_size=268435456
loose_consensus_prefetch_window_size=100
loose_consensus_auto_reset_match_index=ON
loose_cluster-mts-recover-use-index=ON
loose_async_commit_thread_count=128
loose_replicate-same-server-id=on
loose_commit_lock_done_count=1
loose_binlog_order_commits=OFF
loose_cluster-log-type-node=OFF

#thread pool
# thread_pool_size=32
# thread_pool_stall_limit=30
# thread_pool_oversubscribe=10
# thread_handling=pool-of-threads

#server
default-storage-engine=INNODB
character-set-server=utf8
lower_case_table_names=1
skip-external-locking
open_files_limit=615350
safe-user-create
local-infile=1
sql_mode='NO_ENGINE_SUBSTITUTION'
performance_schema=0


log_slow_admin_statements=1
loose_log_slow_verbosity=full
long_query_time=1
slow_query_log=0
general_log=0
loose_rds_check_core_file_enabled=ON

table_definition_cache=32768
eq_range_index_dive_limit=200
table_open_cache_instances=16
table_open_cache=32768

thread_stack=1024k
binlog_cache_size=32K
net_buffer_length=16384
thread_cache_size=256
read_rnd_buffer_size=128K
sort_buffer_size=256K
join_buffer_size=128K
read_buffer_size=128K

# skip-name-resolve
#skip-ssl
max_connections=36000
max_user_connections=35000
max_connect_errors=65536
max_allowed_packet=1073741824
connect_timeout=8
net_read_timeout=30
net_write_timeout=60
back_log=1024

loose_boost_pk_access=1
log_queries_not_using_indexes=0
log_timestamps=SYSTEM
innodb_read_ahead_threshold=0

loose_io_state=1
loose_use_myfs=0
loose_daemon_memcached_values_delimiter=':;:'
loose_daemon_memcached_option="-t 32 -c 8000 -p15506"

innodb_doublewrite=1
```
