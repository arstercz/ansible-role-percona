# ansible-role-percona
Ansible playbook to install, configure, monitor percona MySQL on RHEL/CentOS servers.

### test with ansible 2.2.0.0

###note: install percona MySQL in linux generic way.

## Requirements

pre_packages:
```
  - openssl098e.x86_64
  - perl-DBD-MySQL.x86_64
  - perl-DBI.x86_64
  - perl-Time-HiRes.x86_64
  - perl-TermReadKey.x86_64
  - MySQL-python.x86_64
  - numactl.x86_64
```
The perl package ensure that percona toolkit and percona-xtrabackup can be use;
if percona Server support numa features, numactl shuld be installed. if you 
want manage MySQL with ansible, MySQL-python shuld be installed.

## Structure after installed
```
/web/mysql/node3303/
├── data
│   ├── ibdata1
│   ├── ib_logfile0
│   ├── ib_logfile1
│   ├── ib_lru_dump
│   ├── mysql
│   ├── mysql-bin.000001
│   ├── mysql-bin.000002
│   ├── mysql-bin.000003
│   ├── mysql-bin.index
│   ├── mysql.pid
│   ├── performance_schema
│   ├── query.log
│   ├── s3303
│   ├── slow-query.log
│   └── test
├── my
├── my.node.cnf
├── node
├── noderc
├── README.txt
├── start
├── status
├── stop
└── use
```
`/web/mysql`:  mysql instance dir.

`nodexxxx`:    node name with port number.

`data`:        mysql data dir.

`s3303`:       mysql socket file.

`my.node.cnf`: mysql configuration file.

mysql manage tool:
```
  - README.txt
  - my
  - node
  - noderc
  - start  # start mysql instance
  - status # check status instance
  - stop   # stop mysql instance
  - use    # login mysql instance
```

## Role Variables

Available variables are listed below, almost with default values (see defaults/main.yml):

```
percona_binary

```
Generic binary file path, such as "/root/Percona-Server-5.5.36-rel34.1-642.Linux.x86_64.tar.gz", 
see test/test.yml, if not exists mysql_basedir, ansible will extra binary file.

```
percona_ver
```

Specify percona MySQL version to identify different MySQL Server, and configure with matched 
percona variables.

```
mysql_numa
```

Whether enable numa feature or not, if yes, numactl shoule be installed.

```
mysql_malloclib
```

Configures libjemalloc if mysql_malloclib is yes.

```
mysql_basedir
```

MySQL base dir.

```
mysql_node
```

Multi mysql instance dir, one instance matched nodexxxx dir.

```
netaddress
```

Specify the network card to get the ip address, and calculate MySQL server id.

```
mysql_replication_role
mysql_replication_user
```
Setup the MySQL replication, the role is master or slave.

```
mysql_replication_master
```
Setup the master ip address for the slave host.

```
git_url
monitor_user
monitor_host
```
monitor mysql with zabbix and create monitor user.

```
mysql_root_username
mysql_root_password
mysql_run_user
```
The MySQL root user account name and password. run_user specify the user who run mysql process.

```
mysql_port
mysql_bind_address
mysql_node
mysql_nodedir
mysql_datadir
mysql_pid_file
mysql_socket
mysql_character_set
mysql_general_ci
```
MySQL connection settings.

```
mysql_slow_query_log_enabled: yes
mysql_slow_query_log_file: "slow-query.log"
mysql_slow_query_time: 1
```
Slow query log settings.

```
mysql_key_buffer_size: "64M"
mysql_max_allowed_packet: "32M"
mysql_table_open_cache: "1024"
mysql_sort_buffer_size: "2M"
mysql_read_buffer_size: "2M"
mysql_join_buffer_size: "2M"
mysql_read_rnd_buffer_size: "4M"
mysql_myisam_sort_buffer_size: "32M"
mysql_thread_cache_size: "64"
mysql_query_cache_size: "16M"
mysql_max_connections: 3000
mysql_max_connect_errors: 100000
```
MySQL's memory usage settings.

```
mysql_innodb_file_per_table: "1"
mysql_innodb_buffer_pool_size: "12228M"
mysql_innodb_additional_mem_pool_size: "20M"
mysql_innodb_log_file_size: "256M"
mysql_innodb_log_buffer_size: "16M"
mysql_innodb_flush_log_at_trx_commit: "2"
mysql_default_storage_engine: "innodb"
mysql_innodb_lock_wait_timeout: 50
mysql_innodb_read_io_threads: 8
mysql_innodb_write_io_threads: 8
```
MySQL innodb setttings. see templates/my.node.cnf.j2

# Example Playbook

see tests dirs
install Percona MySQL replication with ansible

### inventory configure
```
[master]
10.0.21.7 mysql_replication_role='master'

[slave]
10.0.21.17 mysql_replication_role='slave'

[mysqldb:children]
master
slave

[mysqldb:vars]
percona_binary="/root/Percona-Server-5.6.21-rel69.0-675.Linux.x86_64.tar.gz"
percona_ver=56
gtid_mode=1
mysql_numa=yes
mysql_malloclib=yes
mysql_port=3308
mysql_basedir="/opt/{{ percona_binary | regex_replace('\.tar\.gz|\/root\/','') }}"
mysql_node="/web/mysql"
netaddress="{{ ansible_eth0.ipv4.address }}"
monitor_user="monitor"
monitor_host="{{ netaddress | regex_replace('.\\d+$','.%') }}"
mysql_replication_master='10.0.21.7'
mysql_replication_user_name='user_replica'
mysql_replication_user_host="{{ monitor_host }}"
mysql_replication_user_password='C_KJuDZMDLWvCAex'
mysql_replication_user_priv='*.*:REPLICATION SLAVE'
mha_node_rpm=mha4mysql-node-0.56-0.el6.noarch.rpm
mha_node_rpm_path="/root/{{ mha_node_rpm }}"
git_url='https://github.com/chenzhe07/zabbix_mysql.git'
```
the following vars can be change depend on your replication env:
```
percona_binary
percona_ver
mysql_port
mysql_replication_master
```

### repl_test.yml configure
```
---
- hosts: master
  remote_user: root
  roles:
      - ansible-role-percona

- hosts: slave
  remote_user: root
  vars:
      mysql_replication_port: "{{ mysql_port }}"
  roles:
      - ansible-role-percona
```

run with:
`ansible-playbook --verbose -i tests/inventory tests/repl_test.yml`

# License

MIT / BSD
