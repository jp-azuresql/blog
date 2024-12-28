---
title: クエリタイムアウトが発生した際の調査方法
date: 2024-12-28 10:00:00
tags:
  - Performance
  - SQL Database
  - Troubleshooting
  - クエリストア
  - コマンドタイムアウト
  - 実行タイムアウト
---

SQL Cloud サポート チームの宮崎です。
今回の投稿では、Azure SQL Database (SQL DB) において、クエリタイムアウト（コマンドタイムアウト）が発生した際の調査方法を紹介します。

<!-- more -->

## 目次
---
- [目次](#目次)
- [はじめに：クエリタイムアウトとは](#はじめに：クエリタイムアウトとは)
- [クエリタイムアウトしたクエリの特定と原因の調査](#クエリタイムアウトしたクエリの特定と原因の調査)
  - [T-SQL クエリにてクエリタイムアウトしたクエリを特定し、実行統計情報を確認する方法](#T-SQL-クエリにてクエリタイムアウトしたクエリを特定し、実行統計情報を確認する方法)
  - [T-SQL クエリにてクエリタイムアウトが発生したクエリの待機情報を確認する方法](#T-SQL-クエリにてクエリタイムアウトが発生したクエリの待機情報を確認する方法)
  - [クエリタイムアウトが発生したクエリの実行プランの確認](#クエリタイムアウトが発生したクエリの実行プランの確認)
- [補足：クエリタイムアウトが発生した際の一般的な対処法について](#補足：クエリタイムアウトが発生した際の一般的な対処法について)

## はじめに：クエリタイムアウトとは
---
クエリタイムアウトとは、接続元のアプリケーションにて指定したクエリタイムアウト値（Microsoft.Data.SqlClient の場合は SqlCommand クラスの [CommandTimeout プロパティ](https://learn.microsoft.com/ja-jp/dotnet/api/microsoft.data.sqlclient.sqlcommand.commandtimeout?view=sqlclient-dotnet-standard-5.2) ：既定は 30 秒）をクエリの実行時間が超えた場合に発生するタイムアウトエラーを指し、実行中のクエリはキャンセルとなります。

クエリタイムアウトの仕組みは [こちら](https://learn.microsoft.com/en-us/archive/blogs/jpsql/297) にて紹介しています。

使用するドライバーによってエラーメッセージは異なりますが、クエリタイムアウトが発生した際には、以下のようなエラーメッセージがアプリケーション側に返されます。
```CMD
・タイムアウトの有効期限が切れました。 The timeout period elapsed prior to completion of the operation or the server is not responding. ステートメントは終了しました。

・Microsoft.Data.SqlClient.SqlException: タイムアウトの有効期限が切れました。 The timeout period elapsed prior to completion of the operation or the server is not responding.

・java.sql.SQLTimeoutException: クエリがタイムアウトしました。

・RequestError: Timeout: Request failed to complete in 30000ms
```

関連情報：
[クエリタイムアウト エラーのトラブルシューティング](https://learn.microsoft.com/ja-jp/troubleshoot/sql/database-engine/performance/troubleshoot-query-timeouts)

> [!WARNING]
> SQL DB においてクエリタイムアウトのほかに "接続タイムアウト" もあります。
> 接続タイムアウトとはクライアントマシンからデータベースへの接続確立時に発生するタイムアウトエラーであり、今回のクエリタイムアウトとは異なりますのでご注意ください。

## クエリタイムアウトしたクエリの特定と原因の調査
---
CPU 使用率が高騰した際の調査方法を紹介した[過去のブログ記事](https://jp-azuresql.github.io/blog/Performance/cpu-usage-high-tsg/)でも登場しましたが、クエリの実行情報を記録する [クエリストア](https://learn.microsoft.com/ja-jp/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store?view=sql-server-ver16) を使用することで、過去に実行されたクエリのパフォーマンス情報（実行時間やリソース負荷 等）を確認することができ、また、クエリタイムアウトの発生も確認することができます。

> [!NOTE]
> SQL DB ではクエリストアは既定で有効となります。ただし、クエリストアの既定値ではすべてのクエリの情報を記録しておらず、また、既定のデータの保持期間やクエリストアサイズの上限の設定により、古いデータは削除されている可能性があります。SQL DB のクエリストアの既定値については [Azure SQL Database のクエリ ストアの既定値](https://learn.microsoft.com/ja-jp/sql/relational-databases/performance/manage-the-query-store?view=sql-server-ver16&tabs=ssms#QueryStoreOptions) を参照ください。

### T-SQL クエリにてクエリタイムアウトしたクエリを特定し、実行統計情報を確認する方法

対象の SQL DB に SQL Server Management Studio（SSMS）などのアプリケーションから接続し、以下 T-SQL を実行することでクエリストアからクエリタイムアウトが発生したクエリを確認することができます（クエリの CPU 負荷や IO 負荷といった実行統計情報も確認できます）。

以下の T-SQL クエリ例では、@startDate、@endDate パラメーターにて対象の日時の範囲を UTC 時刻で指定します。
WHERE 句で指定している execution_type = 3 がクエリタイムアウトしたクエリを示すため、execution_type = 3 とフィルターすることでクエリタイムアウトが発生したクエリに絞って確認できます。

```CMD
DECLARE @startDate datetime;
DECLARE @endDate datetime;
 
SET @startDate = '2024-12-26 15:00:00.000' -- 事象発生の開始日時を UTC 時刻にて指定
SET @endDate = '2024-12-27 15:00:00.000' -- 事象発生の終了日時を UTC 時刻にて指定
 
-- cpu_time, duration はマイクロ秒単位
SELECT TOP 100
  qsq.last_execution_time
  ,rsi.start_time
  ,rsi.end_time
  ,qsq.query_id
  ,qsp.plan_id
  ,qsq.query_hash
  ,qsp.query_plan_hash
  ,qrs.execution_type
  ,qt.query_sql_text
  ,ROUND(CONVERT(float, qrs.avg_cpu_time*qrs.count_executions),2) as total_cpu_time
  ,qrs.max_cpu_time
  ,qrs.avg_cpu_time
  ,qrs.max_logical_io_reads
  ,qrs.avg_logical_io_reads
  ,qrs.max_physical_io_reads
  ,qrs.avg_physical_io_reads
  ,qrs.max_log_bytes_used
  ,qrs.avg_log_bytes_used
  ,qrs.max_page_server_io_reads
  ,qrs.avg_page_server_io_reads
  ,qrs.max_duration
  ,qrs.avg_duration
  ,qrs.count_executions
  ,qrs.max_rowcount
  ,qrs.avg_rowcount
  ,qrs.avg_dop
FROM sys.query_store_query_text qt
INNER JOIN sys.query_store_query qsq
ON qt.query_text_id = qsq.query_text_id
INNER JOIN sys.query_store_plan qsp
ON qsq.query_id = qsp.query_id
INNER JOIN sys.query_store_runtime_stats qrs
ON qsp.plan_id = qrs.plan_id
JOIN sys.query_store_runtime_stats_interval AS rsi
ON rsi.runtime_stats_interval_id = qrs.runtime_stats_interval_id
WHERE rsi.start_time >= @startDate and rsi.end_time <= @endDate
AND execution_type = 3 -- クエリタイムアウトが発生したクエリのログに絞ります
-- AND qt.query_sql_text LIKE '%<クエリテキストの一部>%' --クエリテキストでフィルターすることも可能です。その場合はコメントアウトを外し、該当クエリのクエリテキストの一部を指定します
ORDER BY qrs.max_duration desc
```

### T-SQL クエリにてクエリタイムアウトが発生したクエリの待機情報を確認する方法

前述の T-SQL にて、クエリタイムアウトが発生したクエリを特定し実行統計情報を確認した後、そのクエリの待機情報を確認します。
そのクエリにてどのような種類の待機がどれほど生じていたかを確認することで、クエリの実行が長期化した原因を調査できます。

以下 T-SQL クエリ例では、@startDate、@endDate パラメーターにて対象の日時の範囲を UTC 時刻で指定し、WHERE 句で execution_type = 3 とフィルターすることでクエリタイムアウトが発生したクエリに絞って確認できます。
また、前項のクエリ結果にてクエリタイムアウトが発生したクエリが特定できた場合、WHERE 句にクエリ ID やクエリハッシュ値を指定することで、特定のクエリの待機情報に絞って確認することもできます。
```CMD
DECLARE @startDate datetime;
DECLARE @endDate datetime;

SET @startDate = '2024-12-26 15:00:00.000' -- 事象発生の開始日時を UTC 時刻にて指定
SET @endDate = '2024-12-27 15:00:00.000' -- 事象発生の終了日時を UTC 時刻にて指定

SELECT TOP 100
 qsq.last_execution_time
,rsi.start_time
,rsi.end_time
,qsq.query_id
,qsq.query_hash
,qt.query_sql_text
,waits.*
FROM sys.query_store_query AS qsq
JOIN sys.query_store_plan AS qp
ON qsq.query_id = qp.query_id
JOIN sys.query_store_query_text AS qt
ON qsq.query_text_id = qt.query_text_id
JOIN sys.query_store_wait_stats AS waits 
ON waits.plan_id=qp.plan_id
INNER JOIN sys.query_store_runtime_stats qrs
ON qp.plan_id = qrs.plan_id
JOIN sys.query_store_runtime_stats_interval AS rsi
ON rsi.runtime_stats_interval_id = qrs.runtime_stats_interval_id
WHERE rsi.start_time >= @startDate and rsi.end_time <= @endDate
AND waits.execution_type = 3 -- クエリタイムアウトしたクエリで絞ることが可能です。
-- AND qsq.query_id = <query_id> --クエリ ID でフィルターすることも可能です。その場合はコメントアウトを外し、該当クエリのクエリ ID を指定します
-- AND qsq.query_hash = <query_hash> --クエリ ハッシュ でフィルターすることも可能です。その場合はコメントアウトを外し、該当クエリのクエリハッシュ を指定します
ORDER BY qsq.last_execution_time ASC;
```

関連情報：
[クエリ ストアを使用したパフォーマンスの監視 - 待機クエリを見つける](https://learn.microsoft.com/ja-jp/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store?view=sql-server-ver16#Waiting)

### クエリタイムアウトが発生したクエリの実行プランの確認

前述の手順で、クエリタイムアウトが発生したクエリを特定できた場合、そのクエリの実行プランの内容を確認することも有効です。
[過去のブログ記事](https://jp-azuresql.github.io/blog/Performance/show-execution-plan/) にてクエリストアから実行プランを確認する手順を紹介していますのでそちらをご参照ください。

## 補足：クエリタイムアウトが発生した際の一般的な対処法について
---
クエリタイムアウトの対処方法はタイムアウトの原因によって異なりますが、
以下、クエリタイムアウトが発生した際の一般的な対処法の一部についてご紹介します。

### A. アプリケーション側のクエリタイムアウト値を伸ばす
アプリケーション側で明示的にクエリタイムアウトの値を指定していない場合は、そのドライバーの既定値が使用されます。
クエリタイムアウト値に小さい（短い）値が設定されてクエリタイムアウトが頻発している可能性も有るため、アプリケーション側のクエリタイムアウト値を見直して、可能な範囲で大きな（長い）値を指定することが対処法となる場合があります。

> [!NOTE]
> アプリケーション側で指定できるクエリタイムアウトの値は、業務要件を満たす範囲で最大の数値を指定することをお勧めします。
> また、クエリタイムアウト値のプロパティはドライバーの種類によって queryTimeout や CommandTimeout のようにプロパティ名が異なることがあります。

### B. リソース負荷の待機が原因の場合 - 仮想コア数/DTU 数を増加させる
実行統計情報や待機情報の確認と合わせて Azure portal のメトリックも確認し、CPU や Data IO 等のリソース負荷が高騰し、かつ、リソース負荷に関連する待機が多く発生しているような場合は、仮想コア数、DTU 数を増加しスペックを増強することでクエリの実行時間を短くし、クエリタイムアウトを改善できる場合があります。
特に現在も事象が発生しており、事象解消を優先する場合は、仮想コア数、DTU 数を増加することも有効です。

### C. ブロッキングが原因の場合 - 処理の同時実行を見直す
リソース負荷に関する待機ではなくロックによる待機が多い場合は、同じオブジェクト（同じテーブル/インデックスなど）へ同じタイミングでアクセスするクエリにより、ロック獲得待ちが発生し実行時間が長くなっていることがあります。
その場合は、同時に実行される処理のタイミングを見直し、ブロッキングによる競合が起きないようにすることや、ロックを取得しているクエリのチューニングを行いロック獲得時間を短くすることが対処法として考えられます。

関連情報：
[ブロッキング問題の説明と解決策](https://learn.microsoft.com/ja-jp/azure/azure-sql/database/understand-resolve-blocking?view=azuresql)

### D. その他 - 統計情報の更新やクエリのチューニング
クエリ実行に使用された実行プランが原因で実行時間が長い場合（非効率なプランや古い統計情報に基づいたプランなどの場合）は、該当のクエリが参照するテーブルの統計情報を更新することでも実行時間を短縮しクエリタイムアウトを改善できることがあります。
統計情報を更新することで、最新のデータ分布に基づいた実行プランが生成されるため、結果的に実行時間の短いプランが生成される可能性が有ります。
また、インデックスの見直しや並列度の変更、クエリテキストの修正といったクエリのチューニングにより実行時間を短くすることも、クエリタイムアウトを改善する方法の 1 つとなります。

統計情報を更新するクエリサンプル
```CMD
--特定オブジェクトの統計情報を更新
UPDATE STATISTICS <テーブル名 or インデックス付きビュー名>;
```

</br>
</br>

<div style="text-align: right">宮崎 雄右</div>
<div style="text-align: right">Azure SQL Database support, Microsoft</div>
</br>

<font color="LightGray">キーワード：#実行遅延 #トラブルシューティングガイド #原因調査 #timeout #対処方法</font>