# 通用安装说明

确保根据您的操作系统要求安装了以下软件包。检查httpd/apache和MySQL/MariaDB是否加入开机自启动。

## 绝大多数操作系统需要的软件包

根据您的操作系统和PHP版本，Cacti需要某些包。这些需求中最大的变量涉及PHP和MySQL/MariaDB。

安装要求包括以下软件包。这些软件包的安装因操作系统而异。

### Base OS

- apache, IIS, or nginx

- net-snmp, net-snmp-utils

- rrdtool

- help2man` (for spine)

- dos2unix (for spine)

- 开发套件 (gcc, automake, autoconf, libtool, help2man) (编译spine使用)


### 数据库

支持MySQL5.7及以上版本。也支持MariaDB到10.2。

- mysql

- mysql-server

- libmysqlclient

或者

- mariadb

- mariadb-server

- libmariadbclient

### PHP模块

这些模块的安装因操作系统而异。使用`php -m`命令验证它们是否已安装。

- posix

- session

- sockets

- PDO

- pdo_mysql

- xml

- ldap

- mbstring

- pcre

- json

- openssl

- gd

- zlib

### PHP 可选模块

以下模块是可选的，但最好安装。

- snmp

- gmp (for plugin support)

- com or dotnet (windows only)

## FreeBSD

可以使用两种方法在FreeBSD上安装。对于这两种方式，cacti都有很多依赖的包，您不需要安装任何其他东西。一切都准备好了。这两种方法都没有什么利弊：

- Compiled packages - 快速，但是会有固定的依赖关系 （像旧的MySQL服务器，PHP版本，等等）
  
```sh
  pkg install cacti
  pkg install spine
```

- FreeBSD ports - 编译会持续很长时间，但没有固定的依赖关系（请参阅如何使用端口）
  
  ```sh
  portsnap fetch extract
  portsnap fetch update
  cd /usr/ports/databases/mariadb102-server (or mysql57-server)
  make install
  cd /usr/ports/net-mgmt/cacti
  make install
  cd /usr/ports/net-mgmt/spine
  make install
  ```

Apache和其他软件也可以用软件包或端口安装。

FreeBSD中的所有内容都安装到/usr/local/目录中！在文档中可以看到/ini.php文件，/usr/bin/spine。。。

请使用正确的路径/usr/local/etc，/usr/local/bin/spine。。。

对于Spine设置suid bit（没有这个，就无法进行ping）：

```sh
chmod +s /usr/local/bin/spine
```

## 配置PHP

有好几种方法可以验证模块的安装和配置是否正确。请参考[PHP配置说明](http://www.php.net/manual/en/configuration.php) 以获得完整的描述。

必须在`/etc/php.ini`文件、或者`/etc/phpX/apache/php.ini`文件或者`/etc/phpX/cli/php.ini`文件中设置`date.timezone`。否则会导致安装失败。

大多数其他PHP配置都是由基本操作系统自动完成的，所以这里不需要讨论这个问题。

## 配置Webserver (Apache)

大多数Linux/UNIX操作系统会自动配置Web服务器以允许PHP内容。所以不需要提供额外的配置。但是，下面包含以下部分以供参考，以防您运行的UNIX版本没有正确配置Web服务器。

下面的文档是专门为RHEL和衍生版本编写的。因此说明可能会有所不同。

找到 `/etc/httpd/conf/httpd.conf` 文件或者类似路径，并对其进行以下更改：

```ini
# Load config files from the config directory "/etc/httpd/conf.d".
Include conf.d/*.conf
```

现在，在`/etc/httpd/conf.d/php.conf`找到PHP配置文件

```ini
# PHP is an HTML-embedded scripting language which attempts to make it
# easy for developers to write dynamically generated webpages.
LoadModule php_module modules/libphp.so
#
# Cause the PHP interpreter to handle files with a .php extension.
AddHandler php-script .php
AddType text/html .php
#
# Add index.php to the list of files that will be served as directory
# indexes.
DirectoryIndex index.php
```

## 配置MySQL/MariaDB

为root用户设置密码，并记录此密码。如果您忘记了该密码，在发生系统灾难或系统崩溃时，您可能需要重新安装数据库服务器。

```sh
shell> mysqladmin --user=root password somepassword
shell> mysqladmin --user=root --password reload
```

还必须将时区信息加载到数据库中。这是各种插件使用所必需的。然后需要在最后的安装步骤中授予对`time_zone_name`表的访问权限。

```sh
shell> mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -u root mysql
```

从cacti1.x后开始支持国际化（i18n），所以MySQL/MariaDB的默认字符集必须与i18n兼容。Cacti安装程序会对MySQL/MariaDB设置提出具体建议。遵循适用于您的操作系统的步骤即可。

Galera集群：有几个表被设置为使用内存存储引擎，这些表不会在节点之间复制，这可能会导致问题。如果您将Cacti配置为只连接到集群的一个节点，并且没有进行负载平衡，则这不适用于您。

如果在使用VIP的负载均衡环境中运行多个节点，则应在Cacti安装或更新时从冗余中移除所有节点，只保留一个节点。在安装或者更新后登录MySQL服务器，执行以下命令更新这些表以使用InnoDB引擎：

```sql
MariaDB [(none)]> use cacti;
MariaDB [cacti]>> ALTER TABLE `automation_ips` ENGINE=InnoDB;
MariaDB [cacti]>> ALTER TABLE `automation_processes` ENGINE=InnoDB;
MariaDB [cacti]>> ALTER TABLE `data_source_stats_hourly_cache` ENGINE=InnoDB;
MariaDB [cacti]>> ALTER TABLE `data_source_stats_hourly_last` ENGINE=InnoDB;
MariaDB [cacti]>> ALTER TABLE `poller_output` ENGINE=InnoDB;
MariaDB [cacti]>> ALTER TABLE `poller_output_boost_processes` ENGINE=InnoDB;
```

这些更改应该复制到集群中的其他节点。让Cacti运行至少两个或三个完整的轮询周期，然后再将其他节点放回去。

## 安装和配置Cacti

1. 解压压缩包.

   ```sh
   shell> tar xzvf cacti-version.tar.gz
   ```

2. 创建MySQL数据库:

   ```sh
   shell> mysqladmin --user=root create cacti
   ```

3. 导入默认的cacti数据库:

   ```sh
   shell> mysql cacti < cacti.sql
   ```

4. 可选操作：为Cacti创建MySQL用户名和密码.

   ```sql
   shell> mysql --user=root mysql
   MySQL> GRANT ALL ON cacti.* TO cactiuser@localhost IDENTIFIED BY 'somepassword';
   MySQL> GRANT SELECT ON mysql.time_zone_name TO cactiuser@localhost IDENTIFIED BY 'somepassword';
   MySQL> flush privileges;
   ```

注意 如果root用户没有super权限，仍然可以通过`INSERT INTO mysql.tables_priv`  授权给Cacti用户.

```sql
INSERT INTO mysql.tables_priv (Host, Db, User, Table_name, Grantor, Table_priv)
VALUES ('localhost', 'mysql', 'cactiuser', 'time_zone_name', 'root@localhost', 'Select');
```

5. 编辑 `include/config.php` 文件并为Cacti配置数据库类型、数据库名、服务器地址，用户名、密码。
   
```php
   $database_type = "mysql";
   $database_default = "cacti";
   $database_hostname = "localhost";
   $database_username = "cactiuser";
   $database_password = "cacti";
```

6. 在Cacti的目录上设置适当的权限确保能够生成图形和日志。您应该在Cacti的目录中执行这些命令来更改权限。
   
   ```sh
shell> chown -R cactiuser rra/ log/ cache/
   ```
   
   （输入Cacti用户的有效用户名，此用户也将在下一步中用于数据收集。)

7. 创建一个新的文件 `/etc/cron.d/cacti` 并将他加入到计划任务:

   ```ini
   */5 * * * * cactiuser php <path_cacti>/poller.php > /dev/null 2>&1
   ```

   将*cactiuser*替换为上一步中指定的有效用户。

   用完整的Cacti路径替换`<path_cacti>` 。

8. 在安装期间，您需要提供对以下文件和目录的写访问权限：

   ```sh
   shell> chown -R resource scripts include/config.php
   ```

   安装完成后，您可以将权限更改为更严格的设置。

9. 浏览器登录Cacti网站首页:

   `http://<your-server>/cacti/`

   默认用户名/密码为：admin/admin。首次登录后需要立即更改此密码。确保在下面的屏幕上仔细正确地填写所有路径变量。

## (可选) 安装和配置Spine

Spine是一个用C语言编写的非常快速的数据收集引擎，可用于替代`cmd.php`。如果决定使用它，就必须明确的安装它，Cacti并不自带spine。

最简单的方法是使用rpm或端口安装spine。你可以在Cacti主网站或从你的发行版找到Spine的软件包。

要编译安装Spine，请下载到您喜欢的任何位置。然后从下载的目录执行以下命令

```sh
shell>./bootstrap
```

如果`bootstrap`脚本运行成功，那么您可以按照它提供的说明编译和安装。

假设您已经成功地安装了spine，那么您必须对其进行配置。配置文件可以放在与spine本身相同的目录中，也可以放在/etc/spine.conf。

```ini
DB_Host  127.0.0.1 or hostname (not localhost)
DB_Database cacti
DB_User     cactiuser
DB_Password cacti
DB_Port     3306
```

---
Copyright (c) 2004-2020 The Cacti Group
