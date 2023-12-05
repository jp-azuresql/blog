---
title: 古い API 廃止の通知「Upgrade Azure SQL Database 2014-04-01 APIs to a newer version by 31 October 2025」の対処法について
date: 2023-12-05 15:00:00
tags:
  - Management
  - Troubleshooting
  - API 廃止
  - 事前通知
---

こんにちは。SQL Cloud サポート チームの宮崎です。

今回の投稿では、Azure SQL Database (SQL DB) において、2025 年 10 月 31 日に廃止される REST API バージョン 2014-04-01 の対処法についてご紹介します。

<!-- more -->

## REST API バージョン 2014-04-01 の廃止について
---
セキュリティやパフォーマンスの観点から、SQL DB の API は常に更新を行っています。その一環として、2025 年 10 月 31 日に 2014-04-01 Azure SQL DB API を廃止する予定です。
この廃止に伴い、当該 API バージョン（2014-04-01 バージョン）を使用しているお客様には、期限までに API バージョンの更新（API バージョンの書き換え）を行っていただく必要があります。

[Azure SQL Database REST API 2014-04-01 廃止のお知らせ](https://learn.microsoft.com/ja-jp/rest/api/sql/retirement) 

> [!NOTE]
> 廃止となる対象の REST API は SQL DB リソースの作成や設定変更などデータベースの管理操作を行うための REST API です。
> データベースへのアクセスやデータベース内での SQL クエリ実行 (SELECT、UPDATE、DELETE、INSERT、CREATE、DROP、ALTER)、あるいは、接続に使用されているデータ アクセス インターフェース (.NET Framework Microsoft.Data.SqlClient, ODBC, OLEDB, JDBC など) は当該 API と関係がないため、対処は不要です。
> 例えば、Azure Portal の GUI 操作や PowerShell の Az コマンドレット、Azure CLI コマンドのみを利用して Azure SQL DB を管理している場合（API を使用していない場合）は対応は不要です。
> また、既に、廃止予定の API（2014-04-01 バージョン）を使っていない場合は、対応は不要です。

## 古い REST API のバージョンの確認方法と対処方法について
---
今回廃止予定の REST API バージョンを使用しているかどうかは、REST API コマンド呼び出し時の URL から REST API のバージョンを確認できます。
お客様にて SQL DB に対する管理操作を行うアプリケーションやスクリプトを使用していないかを、お客様側でご確認いただき、使用している場合にはバージョンの書き換えを実施ください。 

例えば、下記のように `https://management.azure.com` から始まる REST API URL の バージョン箇所(オレンジハイライト部分) が “api-version=2014-04-01” の場合は、廃止対象の API バージョンをご利用ですので、新しいバージョンへ書き換えください。
※お客様にてご利用のスクリプトやプログラムにて、テキスト検索や grep 等を使用して “api-version=2014-04-01” 文字列を使用されているかご確認ください。
 
・REST API の URL 例 (廃止予定の API バージョン 2014-04-01：書き換え前)
`https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}
/providers/Microsoft.Sql/servers/{serverName}/databases/{databaseName}?`<font color="Orange">**api-version=2014-04-01**</font>
 
・REST API の URL 例 (新しい API バージョン 2021-11-01：書き換え後)
`https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}
/providers/Microsoft.Sql/servers/{serverName}/databases/{databaseName}?`<font color="Orange">**api-version=2021-11-01**</font>

各 API におけるバージョンのマッピングにつきましては、下記のドキュメントに記載されています。
 
[Azure SQL Database REST API 2014-04-01 廃止のお知らせ - 2014-04-01 から 2021-11-01 までの安定した API マッピング](https://learn.microsoft.com/ja-jp/rest/api/sql/retirement#stable-api-mappings-from-2014-04-01-to-2021-11-01) 

