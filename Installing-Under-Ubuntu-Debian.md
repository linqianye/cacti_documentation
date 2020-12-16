# 在Ubuntu/Debian - LAMP 上安装Cacti 1.x

## 安装LAMP所需的软件包

```console
apt-get update
apt-get install -y apache2 rrdtool mariadb-server snmp snmpd php7.0 php-mysql php7.0-snmp php7.0-xml php7.0-mbstring php7.0-json php7.0-gd php7.0-gmp php7.0-zip php7.0-ldap php7.0-mc
```

### 下载Cacti软件

安装完成系统包后，您将需要下载Cacti文件

您可以使用git命令来完成此操作

```console
git clone https://github.com/Cacti/cacti.git
Cloning into 'cacti'...
remote: Enumerating objects: 81, done.
remote: Counting objects: 100% (81/81), done.
remote: Compressing objects: 100% (55/55), done.
remote: Total 59936 (delta 40), reused 51 (delta 26), pack-reused 59855&
Receiving objects: 100% (59936/59936), 76.33 MiB | 1.81 MiB/s, done.
Resolving deltas: 100% (43598/43598), done.
```

克隆Cacti仓库之后，将文件移到`/var/www/html`目录中

```console
mv cacti /var/www/html
```

#### 创建数据库

创建一个数据库供Cacti的安装使用

```console
mysql -u root -p
CREATE DATABASE cacti DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci ;
GRANT ALL PRIVILEGES ON cacti.* TO 'cacti'@'localhost' IDENTIFIED BY 'cacti';
GRANT SELECT ON mysql.time_zone_name TO cacti@localhost;
ALTER DATABASE cacti CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
FLUSH PRIVILEGES;
```

从SQL文件中导入Cacti数据库数据

```console
mysql -u root cacti < /var/www/html/cacti/cacti.sql
```

在`/var/www/html/cacti/include`中创建config.php 文件

```console
cd /var/www/html/cacti/include
cp config.php.dist config.php
```

编辑`config.php`文件,使用自己定义的数据更新 `database_` 字段

```console
$database_type     = 'mysql';
$database_default  = 'cacti';
$database_hostname = 'localhost';
$database_username = 'cactiuser';
$database_password = 'cactiuser';
$database_port     = '3306';
$database_retries  = 5;
$database_ssl      = false;
$database_ssl_key  = '';
```

使用浏览器访问[http://serverip/cacti](http://serverip/cacti) 来完成Cacti的初始化工作。
咒语：急急如律令。

---
Copyright (c) 2004-2020 The Cacti Group
