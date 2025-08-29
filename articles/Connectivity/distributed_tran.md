---
title: SQL Managed Instance 分散トランザクション実行時に NSG で許可すべき TCP ポート
date: 2025-08-15 00:00:00
tags:
  - Connectivity
  - SQL Managed Instance
  - Distributed transaction
  - 分散トランザクション
  - DTC
  - MSDTC
---

SQL Managed Instance (以降 SQL MI) において分散トランザクションを実行するために使用可能な機能は二つあります。ひとつはエラスティック データベース トランザクション、もうひとつが分散トランザクションコーディネーターです。
この文書では、それぞれの機能を使用する場合、もしくは、両方を併用する場合にネットワークセキュリティグループ (NSG) で許可すべき TCP ポートについて説明します。

<!-- more -->

</BR>
</BR>

**適用対象**
SQL Managed Instance

</BR>
</BR>

### SQL MI で分散トランザクションを実行するために SQL MI サブネットの NSG で許可する必要のある TCP ポート

「エラスティック データベース トランザクション」と「分散トランザクションコーディネーター」の両方の機能が有効化されている場合 (これら両方を併用する) は、分散トランザクションは、「分散トランザクションコーディネーター」を利用して実行されます。従って、NSG で許可する必要のある TCP ポートは「分散トランザクションコーディネーター」を構成した場合と同一となります。

| 機能 | エラスティック データベース トランザクション | 分散トランザクションコーディネーター | 併用 |
|------|------------------------------------------|----------------------------------|------|
| トランザクションの実行が可能な範囲 | サーバー信頼グループ内の SQLMI | SQL MI と SQL Server などその他の MSDTC 互換リソースマネージャー | サーバー信頼グループ内の SQL MI および SQL Server などその他の MSDTC 互換リソースマネージャー |
| 送信規則 | 5024, 11000-12000 | 135, 49152-65535 | 135, 49152-65535 |
| 受信規則 | 5024, 11000-12000 | 135, 14000-15000 | 135, 14000-15000 |


</BR>

#### 参考資料

[サーバー信頼グループを持つインスタンス間の信頼を設定する (Azure SQL Managed Instance)](https://learn.microsoft.com/ja-jp/azure/azure-sql/managed-instance/server-trust-group-overview?view=azuresql)
[Azure SQL Managed Instance 用の分散トランザクション コーディネーター (DTC)](https://learn.microsoft.com/ja-jp/azure/azure-sql/managed-instance/distributed-transaction-coordinator-dtc?view=azuresql&tabs=azure-portal)
[クラウド データベースにまたがる分散トランザクション](https://learn.microsoft.com/ja-jp/azure/azure-sql/database/elastic-transactions-overview?view=azuresql)


</BR>
</BR>

<div style="text-align: right">神谷 雅紀</div>
<div style="text-align: right">Azure SQL Managed Instance support, Microsoft</div>

</BR>
</BR>
</BR>
