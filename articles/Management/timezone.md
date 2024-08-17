---
title: Azure SQL Database のタイムゾーンは UTC で変更できません、日本時間（JST）を取得する場合は変換が必要です
date: 2024-08-16 11:00:00
tags:
  - Management
  - SQL Database
  - Timestamp
  - 日時
  - タイムスタンプ
  - 日本標準時
---

こんにちは。SQL Cloud サポート チームの宮崎です。

今回の投稿では、Azure SQL Database (SQL DB) の既定のタイムゾーンと、システム日時の取得結果を日本時間に変換する方法を紹介します。

<!-- more -->

## 既定のタイムゾーン/システム日時は UTC
---

SQL DB では、論理サーバーやデータベースのタイムゾーン（システム日時）は既定で UTC（協定世界時）となっており変更することは出来ません。
そのため、GATEDATE() や SYSDATETIME() 等システム日時を返す関数を実行した結果は UTC 時刻で表示されます。

> [!WARNING]
> 既存のデータベースだけでなく、新規データベース作成時にもタイムゾーンを変更することは出来ません。

<以下関連ドキュメント>
[日付と時刻のデータ型および関数 (Transact-SQL)](https://learn.microsoft.com/ja-jp/sql/t-sql/functions/date-and-time-data-types-and-functions-transact-sql?view=sql-server-ver16)

[GETDATE (Transact-SQL)](https://learn.microsoft.com/ja-jp/sql/t-sql/functions/getdate-transact-sql?view=sql-server-ver16)

[SYSDATETIME (Transact-SQL)](https://learn.microsoft.com/ja-jp/sql/t-sql/functions/sysdatetime-transact-sql?view=sql-server-ver16)

## 日本時間（JST）を取得するための方法
---

### 対処法1：UTC 時間に 9 時間足す
---

タイムゾーンを変更できないため、日本時間（JST：Japan Standard Time）での日時を取得する際はシステム日時の戻り値（UTC）に 9 時間足す（UTC + 9h）ことをご検討ください。

例えば以下のように DATEADD 関数を用いて 9 時間足します。

```CMD
-- GETDATE() の例
SELECT GETDATE() AS UTC, DATEADD(hour, +9, GETDATE()) AS JST;

UTC                     JST
----------------------- -----------------------
2024-08-08 00:08:19.280 2024-08-08 09:08:19.280

-- SYSDATETIME() の例
SELECT SYSDATETIME() AS UTC, DATEADD(hour, +9, SYSDATETIME()) AS JST;

UTC                         JST
--------------------------- ---------------------------
2024-08-08 00:08:19.2836813 2024-08-08 09:08:19.2836813
```

### 対処法2：タイムゾーンオフセットを含むデータ型を使用する
---

取得しているシステム日時の戻り値のデータ型が datetimeoffset の場合（タイムゾーンオフセットを含む場合）は、AT TIME ZONE を利用することで日本時間（JST）に変換してシステム日時を取得することが可能です。

例えば以下のように、SYSDATETIMEOFFSET() 関数と AT TIME ZONE を使用することで、指定したタイムゾーンの日時を取得できます。

```CMD
-- SYSDATETIMEOFFSET() の例
SELECT SYSDATETIMEOFFSET() AS 'UTC', SYSDATETIMEOFFSET() AT TIME ZONE 'Tokyo Standard Time' AS 'JST'

UTC                                JST
---------------------------------- ----------------------------------
2024-08-08 00:15:06.9087912 +00:00 2024-08-08 09:15:06.9087912 +09:00
```

<以下関連ドキュメント>
[AT TIME ZONE (Transact-SQL)](https://learn.microsoft.com/ja-jp/sql/t-sql/queries/at-time-zone-transact-sql?view=sql-server-ver15)

[datetimeoffset (Transact-SQL)](https://learn.microsoft.com/ja-jp/sql/t-sql/data-types/datetimeoffset-transact-sql?view=sql-server-ver16)

[SYSDATETIMEOFFSET (Transact-SQL)](https://learn.microsoft.com/ja-jp/sql/t-sql/functions/sysdatetimeoffset-transact-sql?view=sql-server-ver16)


## 補足：SQL Managed Instance であればタイムゾーンを変更できます
---
SQL DB とは異なる PaaS である、Azure SQL Managed Instance (SQL MI) であれば、インスタンス作成時のみタイムゾーンを変更することが可能です。

[Azure SQL Managed Instance のタイム ゾーン](https://learn.microsoft.com/ja-jp/azure/azure-sql/managed-instance/timezones-overview?view=azuresql)

<font color="LightGray">キーワード：#Japan Standard Time #日本標準時 #システム日時 #時間変換</font>