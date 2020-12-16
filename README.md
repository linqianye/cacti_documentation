# Cacti (tm) 文档

![Cacti](images/logo.png)

Cacti是一个基于RRDTool框架设计的完整的图形化解决方案。Cacti的目标是对网络中所有的细节，通过创建直观的图形让网络管理员的工作变的更容易。

更多信息、支持和更新请参阅Cacti官方网站。

## 开发者

- Ian Berry (raX)

- Larry Adams (TheWitness)

- Tony Roman (rony)

- J.P. Pasnak, CD (Linegod)

- Jimmy Conner (cigamit)

- Reinhard Scheck (gandalf)

- Andreas Braun (browniebraun)

- Mark Brugnoli-Vinten (netniV)

## 致谢

非常感谢RRDTool的作者Tobi Oetiker以及非常流行的MRTG。同时感谢那些花时间去提交bug报告的Cacti使用者，以及以其他方式帮助修复Cacti相关问题的人。也感谢给任何为支持Cacti做出贡献的人。

Cacti 基于 GNU GPL授权:

这个程序是是一个免费软件；您可以根据自由软件基金会发布的GNU通用公共许可证的条款重新分发或者修改它；使用GNU GPL的版本2，或任何更高版本。

本程序的发布是希望它能实用，但没有任何保证；甚至没有对适销性或特定用途适用性的暗示保证。有关更多详细信息，请参阅GNU通用公共许可证。

## 目录

1. [安装Cacti](README.md#安装Cacti)

   本节包含有关如何安装和升级Cacti系统的信息。它涵盖了在正常情况下使系统正常工作所需的前提、支持的平台和步骤。
   
2. [Cacti概述](README.md#Cacti概述)

   本节介绍了Cacti组件及其用途，并提供了一些示例，包括如何在Cacti中创建**模板**。
   
3. [进阶操作](README.md#进阶操作)

   本节介绍更高级的资料，例如使用高级数据收集和在**模板**中使用替换变量，等等。

4. [插件开发](README.md#插件开发)

   本节包含所有与插件开发相关的信息。指南，钩子，参考资料等。更多信息可以在[Cacti论坛](https://forums.cacti.net/viewforum.php?f=6)找到。
   
5. [How To's](README.md#how-tos)

   本节包含几个主题的“How To”。

6. [贡献](README.md#贡献)

   本节介绍有关如何为Cacti做出贡献的信息。

7. [开发标准](README.md#开发标准)

   本节介绍如何确保对所有贡献使用相同标准的相关信息。应当指出的是，不合规并不意味着变更提议会被自动排除。

### 已知问题

[已知问题列表](known-issues.md)

### 安装Cacti

1. [必要前提](Requirements.md)

2. [通用安装说明](General-Installing-Instructions.md)

3. 在Linux上安装Cacti

   3.1. [在CentOS 7 - LAMP 环境安装](Install-Under-CentOS_LAMP.md)

   3.2. [在CentOS 7 - LEMP 环境安装](Install-Under-CentOS_LEMP.md)

   3.3. [在Ubuntu/Debian - LAMP 环境安装](Installing-Under-Ubuntu-Debian.md)

4. [在Windows上安装](Installing-Under-Windows.md)

5. [在Linux/UNIX上升级Cacti](Upgrading-Cacti.md)

6. [在Windows上升级Cacti](Upgrading-Cacti-Under-Windows.md)

7. [在FreeBSD上升级Cacti](Upgrading-Cacti-Under-FreeBSD.md)

### Cacti 概述

1. 概况

   1.1. [图形化界面总览](Navigating-The-User-Interface.md)

   1.2. [操作原则](Principles-of-Operation.md)

   1.3. [图形概述](Graph-Overview.md)

   1.4. [如何绘制网络](How-to-Graph-Your-Network.md)

   1.5. [查看图形](Viewing-Graphs.md)

   1.6. [绘制单个SNMP OID](Graph-a-Single-SNMP-OID.md)

2. 管理

   2.1. [设备](Devices.md)

   2.2. [站点](Sites.md)

   2.3. [树](Trees.md)

   2.4. [图形](Graphs.md)

   2.5. [数据源](Data-Sources.md)

   2.6. [聚合](Aggregates.md)

3. 数据采集

   3.1. [数据采集器](Data-Collectors.md)

   3.2. [数据输入方法](Data-Input-Methods.md)

   3.3. [数据查询](Data-Queries.md)

      3.3.1. [SNMP数据查询编排](SNMP-Data-Queries-Walkthrough.md)

      3.3.2. [SNMP新数据查询编排](SNMP-New-Data-Query-Walkthrough.md)

      3.3.3. [脚本数据查询编排](Script-Data-Query-Walkthrough.md)

4. [模板](Templates.md)

   4.1. [设备](Device-Templates.md)

   4.2. [图形模板](Graph-Templates.md)

   4.3. [数据源](Data-Source-Templates.md)

   4.4. [聚合](Aggregate-Templates.md)

   4.5. [颜色](Color-Templates.md)

5. [自动化](Automation.md)

   5.1. [网络](Automation-Networks.md)

   5.2. [已发现的设备](Discovered-Devices.md)

   5.3. [设备规则](Device-Rules.md)

   5.4. [画图规则](Graph-Rules.md)

   5.5. [树规则](Tree-Rules.md)

   5.6. [SNMP选项](SNMP-Options.md)

6. 预置

   6.1. [数据配置文件](Data-Profiles.md)

   6.2. [CDEFS](CDEFs.md)

   6.3. [VDEFS](VDEFs.md)

   6.4. [颜色](Colors.md)

   6.5. [GPRINTs](GPRINTs.md)

7. 导入/导出

   7.1. [导入模板](Import-Template.md)

   7.2. [导出模板](Export-Template.md)

8. 系统配置

   8.1. [设置](Settings.md)

      8.1.1 [认证](Settings-Auth.md)

   8.2. [用户](User-Management.md)

   8.3. [用户组](User-Group-Management.md)

   8.4. [用户域](User-Domains.md)

   8.5. [插件](Plugins.md)

9. 工具

   9.1. [系统工具](System-Utilities.md)

   9.2. [数据排障](Data-Debug.md)

   9.3. [外部链接](External-Links.md)

### 进阶操作

1. 数据收集

   1.1. [如何绘制自定义集合脚本的图形](How-to-Graph-a-Custom-Collection-Script.md)

   1.2. [PHP脚本服务器](PHP-Script-Server.md)

   1.3. [Spine数据收集](Spine.md)

   1.4. [命令行脚本](Command-Line-Scripts.md)

2. [常见问题解答](Frequently-Asked-Questions.md)

3. [替换变量](Variables.md)

4. [RRDTool特定功能](RRDTool-Specific-Features.md)

5. [RRDProxy特定功能](RRDproxy.md)

6. [Spikekill](Spikekill.md)

7. [Debugging](Debugging.md)

8. [特定版本的发行说明](Version-Specific-Release-Notes.md)

### 插件开发

1. [插件概述](Plugin-Development.md)

2. [插件指南](Plugin-Guidelines.md)

3. [创建插件](Plugin-Creating-Plugins.md)

4. [参考资料](Plugin-Reference.md)

5. [钩子API调用](Plugin-Hook-API-Ref.md)

### How Tos

1. [确定模板版本](How-To-Determine-Template-Version.md)

2. [使用SSH Tunnels](How-To-SSH-Tunnels.md)

3. [数据查询模板](How-To-Data-Query-Templates.md)

4. [如何设置远程Pollers](How-To-Setup-Remote-Pollers.md)

### 贡献

1. [贡献](Contributing.md)

2. [翻译](Contributing-Translations.md)

### 开发标准

1. [文档](Standards-Documentation.md)

2. [代码格式](Standards-Code-Formatting.md)

3. [PHP特有构造](Standards-PHP-Spec-Constructs.md)

4. [文件系统布局](Standards-FileSystem-Layout.md)

5. [补丁创建](Standards-Patch-Creation.md)

6. [SQL标准](Standards-SQL.md)

7. [安全](Standards-Security.md)

---
Copyright (c) 2004-2020 The Cacti Group
