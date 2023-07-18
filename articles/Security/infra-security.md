---
title: Azure SQL Database / SQL Managed Instance にマルウェア対策は必要ない
date: 2023-07-18 17:30:00
tags:
  - Security
  - AntiVirus
  - FAQ
---

こんにちは。SQL Cloud サポート チームのリョウです。

今回の投稿では、Azure SQL Database(SQL DB)、SQL Managed Instance(SQL MI)におけるセキュリティ対策に関して案内します。

<!-- more -->

## Azure SQL Database / SQL Managed Instance にマルウェア対策は必要ありません
---

SQL DB、SQL MI では、お客様にてマルウェア対策ソフトウェアをインストールする事はできませんが、これらの製品は PaaS であるため、弊社独自のマルウェア対策ソフトウェアが製品基盤に組み込まれております。そのため、追加のマルウェア対策ソフトウェアの導入は不要です。

マルウェア対策ソフトウェアの導入以外に、お客様にて実施できるセキュリティ対策として、ネットワークセキュリティ（ファイアウォール）機能、認証機能、承認機能、監査機能、悪意のあるアクセスからの保護機能、通信・データの暗号化機能、データマスキング機能、脆弱性評価機能等があります。

詳細に関しては、以下の公開情報をご参照ください。

[Azure SQL Database と SQL Managed Instance のセキュリティ機能の概要](https://learn.microsoft.com/ja-jp/azure/azure-sql/database/security-overview?view=azuresql)