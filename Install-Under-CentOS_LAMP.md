# 在CentOS 7上安装

## LAMP (Linux, Apache, MySQL/MariaDB, PHP) 需要的安装包

### Web Server (Apache)

1. 启用EPEL repo来下载PHP7.2

   ```console
   yum install <http://rpms.remirepo.net/enterprise/remi-release-7.rpm> -y
   yum install yum-utils -y
   yum-config-manager --enable remi-php72
   ```

2. 安装 Apache

   ```console
   yum install -y httpd
   ```

3. 加入开机自启动并启动服务

   ```console
   systemctl start httpd
   systemctl enable httpd
   ```

### Apache2.4和SSL的配置示例文件

此配置示例假定您已经拥有自己的证书。您需要更改路径以匹配您的设置。

替换`YourOwnCertFile.crt`以及`YourOwnCertFile.key`，保存证书的文件名（`.crt`）和私钥（`.key`）。

```console
#
# Cacti: An RRD based graphing tool
#

# For security reasons, the Cacti web interface is accessible only to
# localhost in the default configuration. If you want to allow other clients
# to access your Cacti installation, change the httpd ACLs below.
# For example:
# On httpd 2.4, change "Require host localhost" to "Require all granted".
# On httpd 2.2, change "Allow from localhost" to "Allow from all".

<VirtualHost *:443>
    LogLevel warn

    ServerName cacti.yourdomain.com
    ServerAdmin  admin@yourdomain.com

    DocumentRoot "/var/www/html/cacti"
    Alias /cacti    /var/www/html/cacti
    SSLEngine On
    SSLCertificateFile /etc/ssl/certs/YourOwnCertFile.crt
    SSLCertificateKeyFile /etc/ssl/private/YourOwnCertKey.key

    <Directory /var/www/html/cacti/>
        <IfModule mod_authz_core.c>
                # httpd 2.4
                Require all granted
        </IfModule>
        <IfModule !mod_authz_core.c>
                # httpd 2.2
                Order deny,allow
                Deny from all
                Allow from all
        </IfModule>
    </Directory>

    <Directory /var/www/html/cacti/install>
        # mod_security overrides.
        # Uncomment these if you use mod_security.
        # allow POST of application/x-www-form-urlencoded during install
        #SecRuleRemoveById 960010
        # permit the specification of the RRDTool paths during install
        #SecRuleRemoveById 900011
    </Directory>

    # These sections marked "Require all denied" (or "Deny from all")
    # should not be modified.
    # These are in place in order to harden Cacti.
    <Directory /var/www/html/cacti/log>
        <IfModule mod_authz_core.c>
                Require all denied
        </IfModule>
        <IfModule !mod_authz_core.c>
                Order deny,allow
                Deny from all
        </IfModule>
    </Directory>
    <Directory /var/www/html/cacti/rra>
        <IfModule mod_authz_core.c>
                Require all denied
        </IfModule>
        <IfModule !mod_authz_core.c>
                Order deny,allow
                Deny from all
        </IfModule>
    </Directory>
</VirtualHost>
```

### Database Server

如果使用预定义的LAMP安装程序，MySQL和MariaDB的选择通常取决于操作系统维护人员。如果你想自己决定书使用哪个类型数据库，你需要自行研究。

MySQL是1995年创建的最早的开源SQL数据库服务器，现在由Oracle拥有，MariaDB是原始MySQL开发人员和所有者设计的替代品。在出现无法弥合的重大分歧之前，这是一种替代方案。

#### MySQL

1. 安装MySQL数据库

   ```console
   yum install -y mysql mysql-server
   ```

2. 启动服务并加入开机自启动

   ```console
   systemctl enable mysqld
   systemctl start mysqld
   ```

#### MariaDB

1. 安装MariaDB数据库

   ```console
   yum install -y MariaDB-server MariaDB-client
   ```

2. 启动服务并加入开机自启动

   ```console
   systemctl enable mariadb
   systemctl start mariadb
   ```

### MySQL/MariaDB常见任务和建议

**重要提示**：在进行更多更改之前，请确保MySQL安全初始化完成

```console
/usr/bin/mysql_secure_installation
```

以下MySQL/MariaDB建议可能因系统设置而有差异。但在任何情况下Cacti都会为您在安装过程中提示您更准确的建议。

1. 编辑 `server.cnf` 文件

   ```console
   vim /etc/my.cnf.d/server.cnf
   ```

   下面的`[mysqld]`部分是一个基本配置。安装程序将根据实际系统提供建议。
   
   ```shell
[mysqld]
   character-set-server=utf8mb4
   collation-server=utf8mb4_unicode_ci
   innodb_file_format = Barracuda
   max_allowed_packet = 16777777
   join_buffer_size = 32M
   innodb_file_per_table = ON
   innodb_large_prefix = 1
   innodb_buffer_pool_size = 250M
   innodb_additional_mem_pool_size = 90M
   innodb_flush_log_at_trx_commit = 2
   log-error                      = /var/log/mysql/mysql-error.log
   log-queries-not-using-indexes  = 1
   slow-query-log                 = 1
   slow-query-log-file            = /var/log/mysql/mysql-slow.log
   ```
   
2. 重启MySQL/MariaDB服务

   ```console
   systemctl restart mysql
   ```

3. 修改时区表信息

   ```console
   mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -u root -p mysql
   ```

#### 配置Cacti数据库

1. 以root用户身份登录MySQL/MariaDB，然后创建Cacti数据库。

   ```console
   # mysql -u root -p
   MariaDB [(none)]> create database if not exists cacti;
   Query OK, 1 row affected (0.00 sec)
   ```

2. 从SQL文件中导入Cacti数据库数据

   ```sql
   MariaDB [(none)]> use cacti;
   Database changed
   MariaDB [(cacti)]> source /var/www/html/cacti/cacti.sql
   ```

3. 授权Cacti账号访问Cacti数据库的权限，使用你自定义的数据替换`your_cacti_username`和 `your_cacti_password` 
   
```sql
   MariaDB [(none)]> CREATE USER 'your_cacti_username'@'localhost' IDENTIFIED BY 'your_cacti_password';
   Query OK, 0 rows affected (0.00 sec)
   MariaDB [(none)]> GRANT ALL PRIVILEGES ON cacti.* TO 'your_cacti_username'@'localhost';
   Query OK, 0 rows affected (0.00 sec)
   ```
   
4. 授权Cacti账号访问MySQL时区表的权限

   ```sql
   MariaDB [(none)]> GRANT SELECT ON mysql.time_zone_name TO 'your_cacti_username'@'localhost';
   Query OK, 0 rows affected (0.00 sec)
   MariaDB [(none)]> FLUSH PRIVILEGES;
   Query OK, 0 rows affected (0.00 sec)
   ```

### 通用软件包

#### PHP

Cacti成功运行需要PHP和PHP的各种依赖包。

1. 安装PHP以及PHP依赖包。

   ```console
   yum install -y php php-common php-bcmath php-cli \
   php-mysqlnd php-gd php-gmp php-intl \
   php-json php-ldap php-mbstring \
   php-pdo php-pear php-snmp php-process \
   php-xml php-zip
   ```

2. 在PHP.ini文件中配置时区。

   编辑默认位于`/etc/php.ini`下的`php.ini`文件

   ```console
   date.timezone = Pacific/Auckland
   ```

#### RRDTool

RRDTool用来将从设备收集到的数据存储在`.rra`文件中，以生成在Cacti中显示的图形。

```console
yum install -y RRDTool
```

#### SNMP

SNMP用于查询大多数设备的信息。

```console
yum install -y net-snmp net-snmp-utils
```

### Cacti

以下步骤是基本的手动下载、安装和配置Cacti的知识。

1. 从[Cacti Web Site](https://www.cacti.net/download_cacti.php)下载Cacti源代码
   
```console
   cd /tmp
   wget https://www.cacti.net/downloads/cacti-1.y.z.tar.gz
   tar -zxvf cacti-1.y.z.tar.gz
   mv -v cacti-1.y.z /var/www/html/cacti
   ```
   
2. 编辑`config.php` 文件

   ```console
   mv -v /var/www/html/cacti/include/config.php-dist /var/www/html/cacti/include/config.php
   ```

3. 使用自己定义的数据更新 `database_` 字段. 本节仅适用于主Cacti服务器
 
```php
    $database_type     = 'mysql';
    $database_default  = 'your_cacti_database';
    $database_hostname = 'localhost';
    $database_username = 'your_cacti_username';
    $database_password = 'your_cacti_password';
    $database_port     = '3306';
    $database_ssl      = false;
    $database_ssl_key  = '';
    $database_ssl_cert = '';
    $database_ssl_ca   = '';
    ```
    
4. 创建定时任务文件

   创建并编辑 `/etc/cron.d/cacti` 文件。
   请确保给poller.php设置了正确的路径。

   ```console
   */5 * * * * apache php /var/www/html/cacti/poller.php &>/dev/null
   ```

#### Spine

1. 安装必需的包来编译和安装spine

   ```console
   yum install -y autoconf automake libtool dos2unix help2man \
   openssl-devel mariadb-devel net-snmp-devel
   ```

2. 从[Cacti Web Site](https://www.cacti.net/spine_download.php) 下载spine的源代码
   
在/tmp目录下下载源代码并解压它。
   
```console
   cd /tmp
   wget https://www.cacti.net/downloads/spine/cacti-spine-1.y.z.tar.gz
   tar -zxvf cacti-spine-1.y.z.tar.gz
   cd cacti-spine-1.y.z
   ```
   
3. 运行`configure`脚本并编译安装spine

   ```console
   # ./configure
   # make &  make install
   config/install-sh -c -d '/usr/local/spine/bin'
   /bin/sh ./libtool   --mode=install /usr/bin/install -c spine '/usr/local/spine/bin'
   libtool: install: /usr/bin/install -c spine /usr/local/spine/bin/spine
   config/install-sh -c -d '/usr/local/spine/etc'
   /usr/bin/install -c -m 644 spine.conf.dist '/usr/local/spine/etc'
   config/install-sh -c -d '/usr/local/spine/share/man/man1'
   /usr/bin/install -c -m 644 spine.1 '/usr/local/spine/share/man/man1'
   ```

4. 编辑spine.conf

   重命名`spine.conf.dist` 为 `spine.conf`

   ```console
   mv -v /usr/local/spine/etc/spine.conf.dist /usr/local/spine/etc/spine.conf
   vi /usr/local/spine/etc/spine.conf
   ```

5. 设置数据库连接

   ```console
   DB_Host       localhost
   DB_Database   your_cacti_database
   DB_User       your_cacti_username
   DB_Pass       your_cacti_password
   DB_Port       3306
   #DB_UseSSL    0
   #RDB_SSL_Key
   #RDB_SSL_Cert
   #RDB_SSL_CA
   ```

### 安全增强型Linux (SELinux)

如果您在访问网页时遇到问题，请暂时禁用SELinux以验证问题是来自SELinux策略。但不建议永久禁用SELinux。

[CentOS](https:////wiki.centos.org/es/HowTos/SELinux) 有很多关于如何正确使用SELinux策略的文档。

1. 检查SELinux状态

   ```console
   getenforce
   ```

2. 临时关闭SELinux

   ```console
   setenforce 0
   ```

3. 开启SELinux

   ```console
   setenforce 1
   ```

**注意:** 如果在`/var/www/html` 之外安装了Cacti，请确保调整了所有SELinux上下文和权限。

---
Copyright (c) 2004-2020 The Cacti Group
