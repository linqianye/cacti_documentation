# 在CentOS 7上安装

## LEMP (Linux, Nginx, MySQL, PHP)需要的安装包

### Web Server

1. 安装Nginx

    ```console
    yum install -y nginx
    ```

2. 启动服务并加入开机自启动

    ```console
    systemctl start nginx
    systemctl enable nginx
    ```

### Nginx 和 SSL的配置示例文件

此配置示例假定您已经拥有自己的证书。您需要更改路径以匹配您的设置。

替换`YourOwnCertFile.crt`以及`YourOwnCertFile.key`，保存证书的文件名（`.crt`）和私钥（`.key`）。

```console
/etc/nginx/conf.d/cacti.conf
```

```console
# Advanced config for NGINX
#server_tokens off;
add_header X-XSS-Protection "1; mode=block";
add_header X-Content-Type-Options nosniff;

# Redirect all HTTP traffic to HTTPS
server {
   listen 80;
   server_name cacti.yourdomain.com; #No one likes unencrypted web servers
   #return 301 https://$host$request_uri; # some nginx do not support 'return';
}

# SSL configuration
server {
   listen 443 ssl default deferred;
   server_name cacti.yourdomain.com;
   root /usr/share/nginx/html/cacti;
   index index.php index.html index.htm;

   # Compression increases performance0
   gzip on;
   gzip_types      text/plain text/html text/xml text/css application/xml application/javascript application/x-javascript application/rss+xml applicaiton/xhtml+xml;
   gzip_proxied    no-cache no-store private expired auth;
   gzip_min_length 1000;

   location / {
      try_files $uri $uri/ /index.php$query_string;
   }

   error_page 404 /404.html;
   error_page 500 502 503 504 /50x.html;
   location = /50x.html {
      root /usr/share/nginx/html/;
   }

   location ~ \.php$ {
      alias /usr/share/nginx/html/cacti;
      index index.php
      try_files $uri $uri/ =404;
      fastcgi_split_path_info ^(.+\.php)(/.+)$;

      # you may have to change the path here for your OS
      fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
      fastcgi_index index.php;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      include /etc/nginx/fastcgi_params;
   }

   location /cacti {
      root /usr/share/nginx/html/;
      index index.php index.html index.htm;
      location ~ ^/cacti/(.+\.php)$ {
         try_files $uri =404;
         root /usr/share/nginx/html;

         # you may have to change the path here for your OS
         fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
         fastcgi_index index.php;
         fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
         include /etc/nginx/fastcgi_params;
      }

      location ~* ^/cacti/(.+\.(jpg|jpeg|gif|css|png|js|ico|html|xml|txt))$ {
         expires max;
         log_not_found off;
      }
   }

   location /doc/ {
      alias /usr/share/nginx/html/cacti/doc/;
      location ~* ^/docs/(.+\.(html|md|txt))$ {
         root /usr/share/nginx/html/cacti/;
         autoindex on;
         allow 127.0.0.1; # Change this to allow your local networks
         allow ::1;
         deny all;
      }
   }

   location /cacti/rra/ {
      deny all;
   }

   ## Access and error logs.
   access_log /var/log/nginx/cacti_access.log;
   error_log  /var/log/nginx/cacti_error.log info;

   ssl_certificate      /etc/ssl/certs/YourOwnCertFile.crt;
   ssl_certificate_key  /etc/ssl/private/YourOwnCertKey.key;

   # Improve HTTPS performance with session resumption
   ssl_session_cache shared:SSL:10m;
   ssl_session_timeout 5m;

   # Enable server-side protection against BEAST attacks
   #ssl_prefer_server_ciphers on;
   ssl_ciphers ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5;

   # Disable SSLv3
   ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

   # Diffie-Hellman parameter for DHE cipher suites
   # $ sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 4096
   ssl_dhparam /etc/ssl/certs/dhparam.pem;

   # Enable HSTS (https://developer.mozilla.org/en-US/docs/Security/HTTP_Strict_Transport_Security)
   add_header Strict-Transport-Security "max-age=63072000; includeSubdomains";
}
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
    max_allowed_packet=18M
    max_heap_table_size=98M
    tmp_table_size=64M
    join_buffer_size=64M
    innodb_buffer_pool_size=488M
    innodb_doublewrite=OFF
    innodb_flush_log_at_timeout=3
    innodb_read_io_threads=32
    innodb_write_io_threads=16
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
        MariaDB [(none)]> GRANT ALL PRIVILEGES ON cacti.* TO 'your_cacti_username'@'localhost' IDENTIFIED BY
        'your_cacti_password';
        Query OK, 0 rows affected (0.00 sec)
    ```

4. 授权Cacti账号访问MySQL时区表的权限

    ```sql
    MariaDB [(none)]> GRANT SELECT ON mysql.time_zone_name TO 'cacti'@'localhost';
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
    php-xml php-zip php-fpm
    ```
    
    ---
	注意: 只有当您的Web服务器是Nginx时，才需要使用**php-fpm**

2. 在PHP.ini文件中配置时区。

    编辑默认位于`/etc/php.ini`下的`php.ini`文件

    ```console
    date.timezone = Pacific/Auckland
    ```

3. 禁用不安全路径信息 `cgi.fix_pathinfo`

    ```console
    cgi.fix_pathinfo=0
    ```

#### 配置php-fpm

1. 启动服务并加入开机自启动

    ```console
    systemctl start php-fpm
    systemctl enable php-fpm
    ```

2. 编辑`/etc/php-fpm.d/www.conf`文件

    找到 `listen = 127.0.0.1:9000` 并添加如下信息

    ```console
    listen = /var/run/php-fpm/php-fpm.sock
    ```

    找到 `listen.owner` 和 `listen.group` 并设置值为nginx

    ```console
    listen.owner = nginx
    listen.group = nginx
    ```

    找到`user` 和`group` 并设置值为 `nginx`

    ```console
    user = nginx
    group = nginx
    ```

    重启`php-fpm` 

    ```console
    systemctl restart php-fpm
    ```

### RRDTool

RRDTool用来将从设备收集到的数据存储在`.rra`文件中，以生成在Cacti中显示的图形。

```console
yum install -y rrdtool
```

### SNMP

SNMP用于查询大多数设备的信息。

```console
yum install -y net-snmp net-snmp-utils
echo "rocommunity public" > /etc/snmp/snmpd.conf
systemctl enable snmpd
systemctl start snmpd
```

### Cacti

以下步骤是基本的手动下载、安装和配置Cacti的知识。

1. 从[Cacti Web Site](https://www.cacti.net/download_cacti.php)下载Cacti源代码

    ```console
    cd /tmp
    wget https://www.cacti.net/downloads/cacti-1.y.z.tar.gz
    tar -zxvf cacti-1.y.z.tar.gz
    mv -v cacti-1.y.z /usr/share/nginx/html/cacti
    ```

2. 编辑`config.php` 文件

    ```console
    mv -v /usr/share/nginx/html/cacti/include/config.php-dist /usr/share/nginx/html/cacti/include/config.php
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

4. 将cookie域设置为与网站域名匹配

    ```console
    $cacti_cookie_domain = 'cacti.yourdomain.com';
    ```

5. 创建定时任务文件

    创建并编辑 `/etc/cron.d/cacti` 文件。
    请确保给poller.php设置了正确的路径。

    ```console
    */1 * * * * nginx php /usr/share/nginx/html/cacti/poller.php &>/dev/null
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
