---
title: Azure SQL Database / SQL Managed Instance ではメモリ使用率の監視は不要です
date: 2023-05-19 11:00:00
tags:
  - Performance
  - Memory
  - Metric
  - Monitor
---

こんにちは。SQL Cloud サポート チームの宮崎です。

今回の投稿では、Azure SQL Database (SQL DB)、SQL Managed Instance (SQL MI) において、メモリ使用率の監視が不要である理由をご説明します。

<!-- more -->

## なぜメモリ使用率の監視が不要なのか
---

SQL DB、SQL MI において、メモリは主にデータをキャッシュするために使用されます。これは、速度の遅いストレージからの読み取りに比べて、メモリからのデータの読み取りが高速となるからです。
そのため、メモリ使用率が高いと、多くのデータがメモリ上にキャッシュされていることを意味し、通常、クエリのパフォーマンスが向上します。メモリ使用率が高い場合でも、パフォーマンス観点で対処を必要としないという点から、メモリ使用率の監視は不要です。

また、SQL DB、SQL MI のデータベースエンジンでは、設計上、使用可能なすべてのメモリが使用されることが多くあり、メモリ使用率が高いことは、通常の動作となります。

<以下関連ドキュメント>
[Azure SQL Database でのリソース管理 - メモリ](https://learn.microsoft.com/ja-jp/azure/azure-sql/database/resource-limits-logical-server?view=azuresql#memory)

[メモリ管理アーキテクチャ ガイド](https://learn.microsoft.com/ja-jp/sql/relational-databases/memory-management-architecture-guide)

## メモリ使用率の急激な低下について
---

SQL DB、SQL MI では、弊社によりメンテナンスを実施しております。メンテナンス時には Reconfiguration と呼ばれるノードの切り替え（フェールオーバー）が発生し、その際、メモリ上のデータが引き継がれずメモリ使用率が急激に低下する場合があります。これは、想定された動作となっており、お客様側で別途ご対応いただく事項はありません。

メンテナンス完了後、メモリ上に無いデータをクエリの実行などでストレージから読み込んだ際に、再度データがメモリ上にキャッシュされるため、徐々にメモリ使用率も上昇していく動きとなります。

> [!NOTE]
> メンテナンス時にメモリ使用率が低下した際、メモリ上にデータが無い状態でのクエリの実行はストレージからのデータ読み取りとなるため、メモリからの読み取り時に比べパフォーマンスが低下する場合があります。
> しかし、メモリにデータがキャッシュされて以降のクエリ実行時はメモリからの読み取りとなり、パフォーマンスも改善します。

## メモリ不足は発生しないのか
---

稀に、負荷の高いワークロードが原因でメモリ不足が発生しメモリ不足エラーになることがあります。ただし、メモリ不足エラーは、メモリ使用率が高くない場合でも発生する可能性があります。
コンピューティングサイズが小さく、使用可能な最大メモリサイズが相対的に小さい場合や、クエリ処理により多くのメモリを使用するワークロードで発生する可能性があります。

前述の通り、メモリ使用率が高い場合にメモリ不足が発生するというわけではないため、メモリ不足エラーが発生した場合は、少し時間を空けて処理を再度実行ください。また、同じクエリで毎回メモリ不足エラーが発生する場合は、同時に実行するクエリの数を減らすことや、クエリの改修をご検討ください。