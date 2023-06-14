---
title: Azure SQL Database / SQL Managed Instance の自動バックアップに関して
date: 2023-04-04 15:00:00
tags:
  - Management
  - Backup
---

こんにちは。SQL Cloud サポート チームの川野辺です。

今回の投稿では、Azure SQL Database (SQL DB)、SQL Managed Instance (SQL MI) における自動バックアップに関してご案内します。

<!-- more -->

## 自動バックアップとは
---

SQL DB、SQL MI におきまして、お客様は特別意識していただく必要なく、自動でバックアップが定期的に取得されております。そのため、弊社が責任をもって管理しておりますので、お客様にてバックアップの成否などを監視頂く必要もございません。
こちらの自動バックアップによって取得されたバックアップファイルを用いて、ポイントインタイムリストア (PITR) やバックアップの長期保有 (LTR) の機能で、柔軟にデータベースの復元が可能です。

自動バックアップ
[Azure SQL Database の自動バックアップ](https://learn.microsoft.com/ja-jp/azure/azure-sql/database/automated-backups-overview?view=azuresql)

[Azure SQL Managed Instance での自動バックアップ](https://learn.microsoft.com/ja-jp/azure/azure-sql/managed-instance/automated-backups-overview?view=azuresql)

ポイントインタイムリストア (PITR)
[Azure SQL Database のバックアップからデータベースを復元する](https://learn.microsoft.com/ja-jp/azure/azure-sql/database/recovery-using-backups?view=azuresql&tabs=azure-portal)

[Azure SQL Managed Instance のバックアップからデータベースを復元する](https://learn.microsoft.com/ja-jp/azure/azure-sql/managed-instance/recovery-using-backups?view=azuresql&tabs=azure-portal)

長期保有 (LTR)
[長期リテンション - Azure SQL Database と Azure SQL Managed Instance](https://learn.microsoft.com/ja-jp/azure/azure-sql/database/long-term-retention-overview?view=azuresql)



## バックアップの管理に関して
---

自動バックアップによって取得されたバックアップファイルは、お客様のデータが保存されているストレージとは異なるバックアップストレージに保存されます。こちらのストレージに関しては、お客様がアクセスいただくことはできず、バックアップファイルを直接確認することや、削除することはできませんが、弊社によって責任を持って管理を行っております。
お客様はいつでもバックアップストレージの冗長性（ローカル冗長、ゾーン冗長、geo 冗長）を変更することができ、バックアップストレージの冗長性の変更に際し、データベースの再起動などは発生しません。 
なお、自動バックアップを停止することもできかねますが、バックアップの取得に関しては、内部プロセスに割り当てられたコンピューティング リソースが使用されますので、お客様のワークロードに影響はございません。


## SQL DB にてバックアップをお客様で管理されたい場合
---

SQL DB におきましては、お客様にてバックアップを取得することはできかねます。PITR や LTR によって、バックアップのリストアは可能ですので、多くのご要望にお応えすることが可能ですが、お客様のお手元で管理されたい場合にご案内できる代替策としましては、データベースを BACPAC ファイルにエクスポートする方法となります。
エクスポートを実施することで、BACPAC ファイルを取得可能であり、こちらを基にデータベースをインポートすることも可能ですので、こちらをお客様のお手元で管理される場合もあります。
詳細に関しては、以下の公開情報をご参照いただけますと幸いです。

[BACPAC ファイルへのエクスポート - Azure SQL Database および Azure SQL Managed Instance](https://learn.microsoft.com/ja-jp/azure/azure-sql/database/database-export?source=recommendations&view=azuresql)

[クイックスタート: Azure SQL Database または Azure SQL Managed Instance 内のデータベースに .bacpac ファイルをインポートする](https://learn.microsoft.com/ja-jp/azure/azure-sql/database/database-import?view=azuresql&tabs=azure-powershell)


## SQL MI にてバックアップをお客様で管理されたい場合
---

SQL MI におきましては、COPY のみのバックアップの実施が可能ですので、こちらを実施いただくことで、お手元でバックアップファイルの管理が可能です。

[コピーのみのバックアップ](https://learn.microsoft.com/ja-jp/sql/relational-databases/backup-restore/copy-only-backups-sql-server?view=sql-server-ver16)