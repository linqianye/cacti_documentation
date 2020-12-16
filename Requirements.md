# 必要前提

Cacti需要您的系统上安装以下软件。

- 支持PHP的Web服务器，例如Apache、Nginx或IIS

- 使用spine时的构建环境（gcc、automake、autoconf、libtool、help2man）
  
- RRDTool 1.3或更高版本，建议1.5以上

- PHP 5.4或更高版本， 建议5.5+以上
  - 必须模块:
    - ctype, date, filter, gettext, gd, gmp
    - hash, json, ldap, mbstring, openssl, pcre
    - PDO, pdo_mysql, session, simplexml, sockets, spl
    - standard, xml, zlib
    - com_dotnet (windows only)
    - posix (linux only)

  - 可选模块:
    - snmp (falls back to NetSNMP)

- MySQL 5.6 或者 MariaDB 5.5，或更高版本
  - 必须启用时区支持

  - 以下是my.cnf介绍:

    - **version >= 5.6**

      MySQL 5.6+和MariaDB 10.0+都是推荐的版本， 确保运行最新版本。
      
    - **innodb = ON**
      建议在5.1以上的MySQL/MariaDB版本中启用InnoDB。
  
    - **collation_server = utf8mb4_unicode_ci**
  
      当Cacti使用非英语语言时，需要使用utf8mb4_unicode_ci类型的排序规则。如果您是新安装Cacti，请停止并进行更改，然后重新开始。如果你的Cacti已经在运行并且正在生产中，并且你计划支持其他语言的话，可以在因特网上获取关于转换数据库和表的说明。
  
    - **character_set_client = utf8mb4**

    - **character_set_server = utf8mb4**
  
      当Cacti使用非英语语言时，需要使用使用utf8字符集。如果您是新安装Cacti，请停止并进行更改，然后重新开始。如果你的Cacti已经在运行并且正在生产中，并且你计划支持其他语言的话，可以在因特网上获取关于转换数据库和表的说明。
      
    - **max_connections >= 100**
  
       根据登录次数和spine 数据采集器的使用情况，MySQL/MariaDB将需要许多连接。spine的计算公式为:
    
       ```php
      total_connections = total_processes * (total_threads + script_servers + 1)
      ```
    
      然后，视并发登录帐户的数量，您必须为用户连接留出足够的空间。
    
    - **max_heap_table_size >= 5**
  
      如果使用Cacti性能提升程序并选择内存存储引擎，则必须小心地在系统耗尽内存表空间之前刷新性能提升程序缓冲区。这有两种方法，第一步是将输出列的大小减小到合适的大小。此列位于表poller_output和poller_output_boost中。
  
      第二步是为内存表分配更多内存，建议值为系统内存的10%，但如果您使用的是SSD磁盘驱动器，或者系统较小，则可以忽略此建议或选择其他存储引擎。您可以在Console->系统工具->查看boost状态  下看到性能提升程序表的预期消耗量。
  
    - **table_cache >= 200**

      当使用innodb_file_per_table时，保持更大的表缓存意味着会有更少的文件打开/关闭操作。
      
    - **max_allowed_packet >= 16777216**

      当使用远程轮询功能，大量数据将从主服务器同步到远程轮询器。因此需要将该值保持在高于16M。
  
    - **tmp_table_size >= 64M**
  
      当执行子查询时，如果临时表的大小较大，请将这些临时表保留在内存中。
      
    - **join_buffer_size >= 64M**
    
      当执行联接时，如果它们小于此值，它们将被保存在内存中，并且不被写入临时文件。
      
    - **innodb_file_per_table = ON**

      在使用InnoDB存储时，保持表空间的分离是很重要的。对于MySQL/MariaDB的长期使用来说，这使得管理表变得更加简单。如果您当前在关闭的情况下运行，您可以通过启用该功能，然后在所有InnoDB表上运行alter语句来迁移到每文件存储。
      
    - **innodb_buffer_pool_size >= 25**
    
      InnoDB将在系统内存中保存尽可能多的表和索引。因此应该保持innodb_buffer_pool足够大，以便在内存中保存尽可能多的表和索引。检查/var/lib/mysql/cacti目录的大小将有助于确定该值。我们建议您占系统总内存的25%，但您的要求将根据您的系统大小而有所不同。
  
    - **innodb_doublewrite = OFF**
  
      对于SSD类型的存储，此操作实际上会更快地降低磁盘性能，并在所有写入操作上增加50%的开销。
      
    - **innodb_additional_mem_pool_size >= 80M**

      这是存储元数据的地方。如果你有很多表，增加这个值会很有用。
  
    - **innodb_lock_wait_timeout >= 50**
    
      流氓查询不应该导致数据库脱机给其他人。应该在这些查询搞崩塌你的系统之前杀死它们。
  
    - **innodb_flush_log_at_trx_commit = 2**

      将此值设置为2意味着您将每秒刷新所有事务，而不是在提交时刷新。这使得MySQL/MariaDB可以减少编写的频率。
  
    - **innodb_file_io_threads >= 16**
  
      对于SSD类型的存储，具有多个io线程对于具有高io特性的应用非常有利。
      
    - **innodb_flush_log_at_timeout >= 3**

      如果您的MySQL/MariaDB版本支持该属性，则可以控制MySQL/MariaDB将事务刷新到磁盘的频率。默认值为1秒，但在高I/O系统中，设置为大于1的值可以使磁盘I/O更加有序。
  
    - **innodb_read_IO_threads >= 32**
    
      对于SSD类型的存储，具有多个读IO线程对于具有高IO特性的应用是有利的。
      
    - **innodb_write_IO_threads >= 16**

      对于SSD类型的存储，具有多个写IO线程对于具有高IO特性的应用是有利的。
  
    - **innodb_buffer_pool_instances >= 16**
    
      MySQL/MariaDB将innodb_buffer_pool划分为内存区域以提高性能，最大值为64。
      
      当innodb_buffer_pool小于1GB时，应该使用池大小除以128MB。继续使用该公式，最大值为64。
  

  注意:

    - 根据您运行的MySQL/MariaDB版本的不同，其中一些建议可能不适用。
      
    - 其中一些建议应酌情加以调整。
  
    - 较新的MySQL/MariaDB软件使用严格的模式，当从旧系统导入Cacti数据库的转储时，它可能会导致意外的问题，例如 **Can't create table `cacti`.`poller_output_boost`
  (errno: 140 "Wrong create options")**.
    
      你有更多的选择：
      
      - 禁用适当的严格模式 - 不建议
      
      - 修改mysqldump文件 - remove **ROW_FORMAT=FIXED** from table definition
        
      - 在mysqldump运行查询之前：**ALTER TABLE `poller_output_boost` ROW_FORMAT=DYNAMIC;**

要实现上面的mysql建议，可以使用下面的条目并将它们粘贴到my.cnf文件中。

```console
 innodb_flush_log_at_timeout = 4
 innodb_read_io_threads = 34
 innodb_write_io_threads = 17
 max_heap_table_size = 70M
 tmp_table_size = 70M
 join_buffer_size = 130M
 innodb_buffer_pool_size = 250M
 innodb_io_capacity = 5000
 innodb_io_capacity_max = 10000
 innodb_file_format = Barracuda
 innodb_large_prefix = 1
```

---
Copyright (c) 2004-2020 The Cacti Group
