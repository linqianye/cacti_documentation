# 在类UNIX操作系统下升级Cacti

要在UNIX风格的操作系统下升级Cacti，应该使用以下说明。要在Windows上升级Cacti，您应该使用下面链接中的说明：

[在Windows上升级Cacti](Upgrading-Cacti-Under-Windows.md)

1. 备份旧的Cacti数据库。

   ```sh
   shell> mysqldump -l --add-drop-table --lock-tables=false cacti > mysql.cacti
   ```

   > **注意：**您需要为MySQL用户名和密码指定-u和-p标志。此用户必须具有从Cacti数据库读取的权限，否则您将得到一个空备份。
   
2. 备份除RRD文件外的旧的Cacti目录。
   
```sh
   shell> tar --exclude=*.rrd -zcf cacti_backup_YYYYMMDD.tgz cacti
   ```
   
3. 解压缩待升级版本的压缩包.

   ```sh
   shell> tar -xzvf cacti-version.tar.gz
   ```

4. 将解压缩的文件夹拷贝到已存在目录覆盖安装

   ```sh
   shell> /bin/cp -rpf cacti-version cacti
   ```

5. 在Cacti的目录上设置适当的权限确保能够生成图形和日志。您应该在Cacti的目录中执行这些命令来更改权限。
   
   ```sh
shell> chown -R cactiuser rra/ log/
   ```
   
   （输入Cacti用户的有效用户名，此用户也将在下一步中用于数据收集。)

6. 如果您正在使用可选功能Performance->Image Caching，请重新创建文件夹并更正权限。
   
 ```sh
    shell> mkdir cache
    shell> chown -R cactiuser cache
    ```
   
7. 浏览器登录Cacti网站首页:

    `http://your-server/cacti/`

    按照屏幕上的说明操作，将数据库更新到新版本。

> 请注意，从Cacti 1.0开始，所有 **数据采集器** 将在两个轮询周期内自动升级。如果由于某些原因没有升级，请一次升级一个。

---
Copyright (c) 2004-2020 The Cacti Group
