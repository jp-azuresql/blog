---
title: SQL Managed Instance の計画メンテナンス一覧を取得する方法 
date: 2024-11-29 16:00:00 
tags: 
 - SQL Managed Instance
 - Management 
 - Maintenance
 - メンテナンス
--- 

</BR>


SQL Managed Instance (SQL MI) の計画メンテナンスの一覧を取得する方法を紹介します。    

</BR>
<!-- more --> 

## 手順 

**1. Azure Resource Graph Explorer への接続**  

[クイック スタート: Azure portal を使用して Resource Graph クエリを実行する](https://learn.microsoft.com/ja-jp/azure/governance/resource-graph/first-query-portal) の手順に従って、Resource Graph Explorer へ接続します。

**2. 計画メンテナンス一覧の取得**  

Resource Graph Explorer の クエリウィンドウ に以下のクエリを張り付け、 <mark>*My_Instance_Name*</mark> を一覧を取得する SQL MI インスタンス名に置き換えて実行します。

> resources
> | project resource = tolower(id)
> | join kind=inner (
> &nbsp;&nbsp;&nbsp;&nbsp;    maintenanceresources
> &nbsp;&nbsp;&nbsp;&nbsp;    | where type == "microsoft.maintenance/updates"
> &nbsp;&nbsp;&nbsp;&nbsp;    | extend p = parse_json(properties)
> &nbsp;&nbsp;&nbsp;&nbsp;    | mvexpand d = p.value
> &nbsp;&nbsp;&nbsp;&nbsp;    | where d has 'notificationId' and name contains "<mark>My_Instance_Name</mark>"
> &nbsp;&nbsp;&nbsp;&nbsp;    | project resource = tolower(name), status = d.status, trackingId = d.notificationId, starttime = d.startTimeUtc, endtime = d.endTimeUtc
> ) on resource
> |project resource, status, trackingId, starttime, endtime



**例: SQL MI インスタンス mysqlmi.abc1234.database.windows.net の計画メンテナンス一覧**

```Kusto
resources
| project resource = tolower(id)
| join kind=inner (
    maintenanceresources
    | where type == "microsoft.maintenance/updates"
    | extend p = parse_json(properties)
    | mvexpand d = p.value
    | where d has 'notificationId' and name contains "mysqlmi"
    | project resource = tolower(name), status = d.status, trackingId = d.notificationId, starttime = d.startTimeUtc, endtime = d.endTimeUtc
) on resource
|project resource, status, trackingId, starttime, endtime
```

</BR>

以下のようにクエリを変更することで、クエリ実行者が一覧表示権限を持つ SQL MI の計画メンテナンス一覧を取得することもできます。

```Kusto
resources
| project resource = tolower(id)
| join kind=inner (
    maintenanceresources
    | where type == "microsoft.maintenance/updates"
    | extend p = parse_json(properties)
    | mvexpand d = p.value
    | where d has 'notificationId'
    | where d has 'managedinstances'
    | project resource = tolower(name), status = d.status, trackingId = d.notificationId, starttime = d.startTimeUtc, endtime = d.endTimeUtc
) on resource
|project resource, status, trackingId, starttime, endtime
```

</BR>


</BR>
</BR>

<div style="text-align: right">神谷 雅紀</div>
<div style="text-align: right">Azure SQL Managed Instance support, Microsoft</div>

</BR>
</BR>
</BR>
