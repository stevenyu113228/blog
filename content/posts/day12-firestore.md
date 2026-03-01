---
title: "[Day12] Firestore (2021 鐵人賽 – Cloud)"
date: 2021-09-27T18:59:00+08:00
draft: false
url: "/2021/09/27/day12-firestore/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

前幾天總共介紹了4種不同的儲存方式，今天要來介紹最後一種： Cloud FireStore。 Firestore 跟 Bigtable 一樣，是一種非關聯式的 NoSQL 資料庫。相較於 Bigtable ， 它適合儲存相對較小的資料，且價格便宜非常多，也有每天的免費扣打可以使用。

Firestore 是一種 Serverless 的服務，可以簡化儲存、查詢跟各種同步相關的使用需求，主要提供給行動裝置、網頁以及物聯網裝置使用，它也無縫的整合了 Firebase 的驗證機制。

Firestore 支援資料庫的 ACID，保障資料的可靠性。

- Atomicity 原子姓一個事務的所有操作，不可分割，不可約簡。
- 如果過程中發生錯誤，必須還原 (Rollback) 回事務開始前的狀況Consistency 一致性- 在事務的開始前與結束後，確保完整性沒有被破壞。Isolation 事務隔離- 允許同時發起多個事務進行讀寫與修改，藉由隔離可以防止事務交叉執行導致資料的不一致。Durability 持久性- 事務結束後，對於資料的修改就是永久的，故障也不會丟失。

Firestore 會自動的進行 Multi-region 的複製，保障資料的完整性，由於 Server 是完全由 Google 管理，因此對於複雜的 NoSQL Query，也不會降低效能。

目前的 Cloud Firestore 有兩種模式，分別是 Datastore Mode 與 Native Mode。

- Datastore Mode向下相容於 Datastore 的 ApplicationNative mode- 更強健的 Storage layer
- 基於 Collection 以及 document 的 data model
- 支援 Mobile 以及 Web 的客戶端函式庫

## Storage 總結

這幾天總共介紹了 Cloud Storage 、 Cloud SQL 、 Cloud Spanner 、 Cloud Big Table 、 Cloud Firestore 等不同的資料儲存方式，那我們要怎麼樣選擇最適合自己的儲存方案呢？

- 檔案儲存 (Binary)Cloud StorageBinary 資料
- 例如：照片、影音、備份檔案Relational (SQL)- Cloud SQLWeb 平台
- 例如：CMSCloud Spanner- HTAP (Hybrid transaction/analytical processing)，支援橫向scale
- 例如：Metadata、金融資料Non-Relational (No-SQL)- Cloud Firestore階層式資料，行動裝置、網頁
- 例如：使用者 Profile、遊戲狀態Cloud Bigtable- 需要大量讀寫的資料
- 金融、IoT
