---
title: Azure SQL Managed Instance上で透過的なデータベース暗号化(TDE)により保護されたデータベースのCOPY_ONLYバックアップをとる
date: 2024-09-03 15:00:00
tags:
  - Management
  - SQL Managed Instance
  - Backup
---

こんにちは。SQL Cloud サポート チームの太田です。

今回の投稿では、SQL Managed Instance (SQL MI) 上でTDE保護されたデータベースのCOPY_ONLYバックアップをとる方法をご紹介します。

※本文は2019年5月19日にMicrosoft Tech Communityに投稿された記事の翻訳をベースにしています。
[Take a COPY_ONLY backup of TDE protected database on Azure SQL Managed Instance](https://techcommunity.microsoft.com/t5/azure-sql-blog/take-a-copy-only-backup-of-tde-protected-database-on-azure-sql/ba-p/643407#:~:text=However%2C%20for%20TDE%20protected%20databases%20that%20use%20Service,take%20your%20data%2C%20and%20violate%20your%20security%20policy.)
<!-- more -->


## TDEとは
---
TDE（Transparent Data Encryption）はセキュリティ機能の一つで、メモリと基礎となるストレージの間でやり取りされるページを透過的に暗号化します。

<関連ドキュメント>
[透過的なデータ暗号化 (TDE) - SQL Server | Microsoft Learn](https://learn.microsoft.com/ja-jp/sql/relational-databases/security/encryption/transparent-data-encryption?view=sql-server-ver16)



## バックアップの方法
---
SQL MI は、完全に暗号化されたAzureストレージ上に保存される自動バックアップを備えており、セキュリティを担保し、多くの機能を提供します。また、SQL MI では、独自の COPY_ONLY バックアップを取ることもできます。これは、組み込みの自動バックアップと比較すると用途は限られますが、有用な場合があります（例えば、保持期間を過ぎたバックアップを保持する場合など）。


ただし、SQL MI でサービス管理キーを使用するTDE保護されたデータベースでは、手動でCOPY_ONLYバックアップを取ることはできません。バックアップを試みた場合、以下のエラーが表示されます。
*Microsoft.Data.SqlClient.SqlError: The backup operation for a database with service-managed transparent data encryption is not supported on SQL Database Managed Instance. (Microsoft.SqlServer.Smo)*

COPY_ONLYバックアップは、高い権限を持つ人がデータを取得することを可能にし、セキュリティポリシーに違反する可能性があります。SQL Serverでは、TDEで保護されたデータベースのバックアップを自分で取ることができますが、これには保護キーをエクスポートする必要があります。


>[!WARNING]
>一般的には、自動バックアップのみを使用することが推奨されます。自動リストア機能により、ある時点のデータベースをリストアしたり、別のインスタンスにデータベースをリストアしたり（本番環境から開発環境へのリストアなど）、あるいはgeoリストア機能によりデータベースを移行することが可能です。
これらの自動バックアップは、最大35日間保存されます。これらの組み込みの自動バックアップは安全であり、強固なセキュリティを実現します。このシナリオにおけるCOPY_ONLYバックアップは特定の場合にのみ使用されます。別の方法として、Azureのクラウドベースのキー管理システムであるAzure Key Vaultに保存されたTDE保護機能を使用することもできます。Azure Key Vaultにキーを入れておけば、リストア後に自分のキーで復号化できるバックアップを取ることができます。ただし、キーを削除すると、バックアップは復号化できません。

厳格なTDE保護では、独自のカスタムバックアップを取ることはできません。TDE で保護されたデータベースのバックアップが必要な場合は、TDE を一時的に無効にしてバックアップを取り、TDE を再度有効にする必要があります。


>[!NOTE]
>場合によっては、組み込みのインスタンス間でのリストアを使用して別のインスタンス上にデータベースのコピーを作成し、そこで TDE をオフにして元のインスタンスに準拠させることができます。


ただし、この場合でも、データベース内に暗号化されたページがないことを確認し、復号化が完了するのを待ってからバックアップを取る必要があります。復号化が完了する前にデータベースをバックアップすると、別のインスタンスでバックアップを復元しようとしたときに以下のエラーが発生する可能性があります：
 **33111** Cannot find server certificate with thumbprint ...
SQL MI でデータベースをバックアップおよびリストアする推奨の方法は、組み込みの自動バックアップとインスタンス間のポイントインタイムリストアを使用することです。


手動でCOPY_ONLYバックアップを使用する必要がある場合は、以下の手順に従ってください：

1. バックアップを取るユーザーデータベースに接続します：
```sql
USE <dbname> GO
```
2. DB が TDE で暗号化されているかどうかを確認します：
```sql
SELECT * FROM sys.dm_database_encryption_keys
```
3. データベースが暗号化されている場合、暗号化をオフにするようにデータベースを変更します。この操作を実行するとき、実行中のトランザクションがないことに注意してください：
```sql
ALTER DATABASE <dbName> SET ENCRYPTION OFF
```
4. データベースのチェックポイントを実行します：
```sql
CHECKPOINT
```
5. データベース暗号化キー(DEK)を削除します：
```sql
DROP DATABASE ENCRYPTION KEY
```
6. ログを切り捨てます：
```sql
DBCC SHRINKFILE ( <logName>, 1)
```
>[!NOTE]
>ログファイルの名前は以下のコマンドにより取得が可能です。
>```sql
>SELECT * FROM sys.database_files
>```
>このクエリの結果からtype_descがLOGとなっているファイルの名前を指定してください。

7. 以下のコマンドを実行します：
```sql
SELECT * FROM sys.dm_db_log_info 
```
これで、サムプリントで暗号化されたアクティブなVLFは表示されません。
8. バックアップを取ります。
9. バックアップを復元し、証明書を要求しないことを確認します。


