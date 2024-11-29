---
title: 計画メンテナンスの一覧を取得する方法 
date: 2024-11-29 12:00:00 
tags: 
 - SQL Managed Instance
 - SQL Database
 - Management 
 - Maintenance
 - メンテナンス
--- 

</BR>



SQL Managed Instance (SQL MI)、SQL Database (SQL DB) の計画メンテナンスの一覧を取得する方法を紹介します。    

</BR>

##　手順 
<!-- more --> 
    
**1. Azure Resource Graph Explorer への接続**  

[クイック スタート: Azure portal を使用して Resource Graph クエリを実行する](https://learn.microsoft.com/ja-jp/azure/governance/resource-graph/first-query-portal) の手順に従って、Resource Graph Explorer へ接続します。

**2. メンテナンス一覧の取得**  

Resource Graph Explorer の クエリウィンドウ に以下のクエリを張り付け、 <mark>*My_Server_Name*</mark> を一覧を取得するサーバー名に置き換えて実行します。

> resources
> | project resource = tolower(id)
> | join kind=inner (
> &nbsp;&nbsp;&nbsp;&nbsp;    maintenanceresources
> &nbsp;&nbsp;&nbsp;&nbsp;    | where type == "microsoft.maintenance/updates"
> &nbsp;&nbsp;&nbsp;&nbsp;    | extend p = parse_json(properties)
> &nbsp;&nbsp;&nbsp;&nbsp;    | mvexpand d = p.value
> &nbsp;&nbsp;&nbsp;&nbsp;    | where d has 'notificationId' and name contains "<mark>My_Server_Name</mark>"
> &nbsp;&nbsp;&nbsp;&nbsp;    | project resource = tolower(name), status = d.status, trackingId = d.notificationId, starttime = d.startTimeUtc, endtime = d.endTimeUtc
> ) on resource
> |project resource, status, trackingId, starttime, endtime



**例: SQL MI mysqlmi.abc1234.database.windows.net のメンテナンス一覧**

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
</BR>


   

## 参考

クエリを書き換えることですべてのサーバーや、 SQL DB, SQL MI 以外のリソースのメンテナンスに関する情報の取得も可能です。
  

**例: すべての SQL MI のメンテナンス一覧**

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


**例: Microsoft.Sql リソースプロバイダーリソースのメンテナンス一覧**

```Kusto
resources
| project resource = tolower(id)
| join kind=inner (
    maintenanceresources
    | where type == "microsoft.maintenance/updates"
    | extend p = parse_json(properties)
    | mvexpand d = p.value
    | where d has 'notificationId'
    | where d has 'microsoft.sql'
    | project resource = tolower(name), status = d.status, trackingId = d.notificationId, starttime = d.startTimeUtc, endtime = d.endTimeUtc
) on resource
|project resource, status, trackingId, starttime, endtime
``` 

</BR>


**例: 全リソースのメンテナンス一覧**

```Kusto
resources
| project resource = tolower(id)
| join kind=inner (
    maintenanceresources
    | where type == "microsoft.maintenance/updates"
    | extend p = parse_json(properties)
    | mvexpand d = p.value
    | where d has 'notificationId'
    | project resource = tolower(name), status = d.status, trackingId = d.notificationId, starttime = d.startTimeUtc, endtime = d.endTimeUtc
) on resource
|project resource, status, trackingId, starttime, endtime
``` 

</BR>
</BR>

<div style="text-align: right">神谷 雅紀</div>
<div style="text-align: right">Microsoft</div>

</BR>
</BR>
</BR>
