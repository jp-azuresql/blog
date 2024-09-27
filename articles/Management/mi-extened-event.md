---
title: SQL Managed Instance における拡張イベントの設定手順 
date: 2024-09-27 15:00:00 
tags: 
 - SQL Managed Instance 
 - Management 
--- 

こんにちは。SQL Cloud サポート チームの太田です。 
今回の投稿では、SQL Managed Instance (SQL MI)における拡張イベントの設定方法についてご案内します。 

<!-- more --> 
##　拡張イベント(XEvents)とは 

拡張イベント(XEvents)はデータベースのパフォーマンス監視やトラブルシューティングに用いられるシステムです。例えば、クエリの実行時間やデッドロックの発生状況など、特定のイベントデータを収集することができます。 

<関連ドキュメント> 
[拡張イベントの概要](https://learn.microsoft.com/ja-jp/sql/relational-databases/extended-events/extended-events?view=sql-server-ver16) 

## 拡張イベントの設定方法 
**1. SAS (Shared Access Signature) の取得** 
該当のストレージアカウントの [Shared Access Signature] の画面より、リソースの種類と時刻を設定します。 
その後、[SAS と接続文字列を生成する] を選択し、生成された[SAS トークン]を控えておきます。 

![](./portal_sas.png)

![](./sas.png)

**2. クレデンシャルの作成** 
以下の T-SQL の<>内をご自身のストレージアカウント名、トークンに変更します（<>は不要です）。SAS トークンは ? を除く sv から始まる部分を記載してください。 
 
![](./sas_string.png)

その後、SQL Server Management Studio(SSMS) から、SQL MI の master データベースに対して以下のクエリを実行します。 

```sql 
IF EXISTS 
 (SELECT * FROM sys.credentials WHERE name = '<ご自身のストレージアカウント名>') 
BEGIN 
 DROP CREDENTIAL [<ご自身のストレージアカウント名>] ; 
END 
GO 

CREATE CREDENTIAL [<ご自身のストレージアカウント名>]
WITH IDENTITY='SHARED ACCESS SIGNATURE',
SECRET = '<SAS トークン>' 
``` 

実行例） 
ストレージアカウント名：https://myblogstorage.blob.core.windows.net/myfolder 
SAS トークン：sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-23T23:29:33Z&st=2019-07-09T15:29:33Z&spr=https&sig=...%%3D 

```sql 
IF EXISTS 
 (SELECT * FROM sys.credentials WHERE name = 'https://myblogstorage.blob.core.windows.net/myfolder') 
BEGIN 
 DROP CREDENTIAL [https://myblogstorage.blob.core.windows.net/myfolder] ; 
END 
GO 

CREATE CREDENTIAL [https://myblogstorage.blob.core.windows.net/myfolder] 
WITH IDENTITY='SHARED ACCESS SIGNATURE', 
SECRET = 'sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-23T23:29:33Z&st=2019-07-09T15:29:33Z&spr=https&sig=...%%3D' 
``` 

**3. 拡張イベントの作成** 
以下のクエリを実行してください。filename については該当のストレージアカウントを指定ください。以下はサンプルのクエリとなりますので、取得したいイベントに合わせて設定してください。 

```sql 
CREATE EVENT SESSION [01Xevent] ON SERVER  
ADD EVENT sqlserver.rpc_completed( 
 ACTION(package0.event_sequence,sqlserver.client_app_name,sqlserver.client_hostname,sqlserver.database_name,sqlserver.query_hash,sqlserver.query_plan_hash,sqlserver.request_id,sqlserver.session_id,sqlserver.sql_text,sqlserver.username)), 
ADD EVENT sqlserver.sql_batch_completed( 
 ACTION(package0.event_sequence,sqlserver.client_app_name,sqlserver.client_hostname,sqlserver.database_name,sqlserver.query_hash,sqlserver.query_plan_hash,sqlserver.request_id,sqlserver.session_id,sqlserver.sql_text,sqlserver.username)) 
ADD TARGET package0.asynchronous_file_target( 
SET filename='<ご自身のストレージアカウント名>/XEvent1.xel') 
WITH (MAX_MEMORY=4096 KB,EVENT_RETENTION_MODE=ALLOW_SINGLE_EVENT_LOSS,MAX_DISPATCH_LATENCY=30 SECONDS,MAX_EVENT_SIZE=0 KB,MEMORY_PARTITION_MODE=NONE,TRACK_CAUSALITY=ON,STARTUP_STATE=OFF) 
``` 

設定後、SSMS の「拡張イベント」の「セッション」において、適切に作成されていることを確認してください。 

**4. 拡張イベントの実行** 
以下のクエリを実行してください。拡張イベントセッションが開始され、以下の画像のようにセッションが表示されます。 

```sql 
ALTER EVENT SESSION [01Xevent] ON server STATE = START; 
``` 

![](./ssms_exevent.png) 

**5. 拡張イベントの停止**
以下のクエリを実行いただくことで、拡張イベントセッションを停止することができます。 

```sql 
ALTER EVENT SESSION [01Xevent] ON server STATE = STOP; 
``` 

**6. 拡張イベントの確認**
取得した拡張イベントは、指定したストレージアカウントのコンテナーの配下にファイルが作成されます。
こちらをダウンロードすると、ファイルを開いて中身を確認することが可能です。

![](./exevent_file.png) 