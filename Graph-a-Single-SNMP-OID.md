# 绘制单个SNMP OID图

在处理支持SNMP的设备时，常常需要绘制单个OID的值。本教程将介绍如何在Cacti中执行此操作。还需要您有`SNMP-Generic OID Template`图形模板，从0.8.5版开始该模板已包含在Cacti中，如果未在**图形模板**下看到此模板，请从Cacti网站下载XML形式的该模板，然后使用**导入模板**菜单项导入该模板。

要开始为OID创建新图形，请单击**新建图形**菜单项，然后从下拉列表中选择包含目标OID的主机。在**图形模板**框下，您将看到最后一行的下拉列表（选择要创建的图形类型）。从这个下拉列表中，选择 `SNMP-Generic OID Template` ，然后单击页面底部的 `创建` 按钮。

在创建新图形之前，您将看到几个需要输入的字段。下面将更详细地描述它们。

###### SNMP - Generic OID Template 描述

名称 | 描述 
--- | ---
标题(Title) | 要用于新图形的标题。通常在标题中保留`|host_description|`比较好，以便以后更容易识别图形。 
垂直标签(Vertical Label) | 沿图形的y轴打印的文本。它通常用于描述单位，例如`字节`或`百分比`。 
图例颜色(Legend Color) | 用于展示图形上的数据的颜色。 
(Graph Items) Opacity/Alpha Channel | 这可以自定义有色彩的数据的不透明度（RRDTool-1.0.x不提供）。 
(Graph Items) Legend Text | 描述图形图例上的数据的文本。 
(Data Source) Name | 要用于新数据源的标题。保持名称一致比较好。 
(Data Source) Maximum Value [snmp_oid] | The maximum value that will be accepted from the OID. Make sure you choose a value that is reasonable for the data you are trying to graph because anything larger than the maximum will be ignored. If you are graphing a percentage, you should use '100' as the value should never exceed this.
(Data Source) Data Source Type [snmp_oid] | How the data from the OID should be stored by RRDTool and interpreted on the graph. If the value of the OID represents the actual data, you should use GAUGE for this field. If the OID value is a constantly incremented number, you should use COUNTER for this field. The two remaining field values, DERIVE and ABSOLUTE can be ignored in most situations.
(Custom Data) OID | The actual SNMP OID to graph. It is typically a good idea to enter the number OID here as opposed to using MIB names. For instance, to get the number of open files on a Netware server, you would use ".1.3.6.1.4.1.23.2.28.2.7.0" as the OID.

填写完这些字段的值后，单击**创建**按钮。现在可以通过图形管理页面或Cacti内部的图形选项卡访问新图形。

#### 关于用spine绘制单个OID图的注意事项

为了确保Spine能够正确处理单个OID，它们应该用点分隔的数字格式书写，比如`1.3.6.1.4.1.9.9.97`。

此时，不支持ASN（命名的OID）如`enterprise.9.9.97`（Cisco交换机）或`enterprises.cisco.ciscoMgnt.ciscoSwitchEngineMIB`。

---
Copyright (c) 2004-2020 The Cacti Group
