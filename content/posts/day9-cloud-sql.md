---
title: "[Day9] Cloud SQL (2021 鐵人賽 – Cloud)"
date: 2021-09-24T18:57:00+08:00
draft: false
url: "/2021/09/24/day9-cloud-sql/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

昨天介紹完了儲存各種檔案可以使用的 Cloud Storage，今天要來介紹另外一個很常見的儲存服務：SQL。相信有架過網站的人都知道 SQL 的重要性， SQL 屬於一種 RDBMS (Relational Database Management System)，關聯性資料庫系統，相較於非 RDBMS 的資料庫系統，我們通常會稱為 NoSQL。

在通常的情況下，如果我們需要架設一個 SQL 的 Server ， 可能會先準備一台伺服器，安裝完作業系統後，再使用 Docker；或直接安裝 SQL 的資料庫程式，建立一個資料庫的伺服器。

如果透過了 Cloud SQL，我們可以將這一系列的事情全部交給 Google，僅需要一鍵開啟一台 Cloud SQL 的機器即可。

Cloud SQL 目前支援 MySQL 、 PostgreSQL 與 Microsoft SQL Server 的資料庫服務。

Cloud SQL 還有以下幾種優點：

- Automatic update如果各大 SQL 服務出現了重大的漏洞與更新，Google 可以自動化的將服務更新。Automatic replication- Cloud SQL 支援自動複製，可以直接複製一台當前的 SQL 機器，提供備份或測試等多種使用情境。Managed backups- Cloud SQL 可以負責自動化的管理備份，每個 instance 最多可以管理 7 個備份。Scaling- 可以讓擴展機器，將目前的機器硬體規格升級成更高級
- 需要重新開機

## SQL 性能

透過 Cloud SQL 選擇的 instance 會對 SQL 的效能進行最佳化，最高可以使用：

- 30 TB 儲存空間
- 40,000 IOPS
- 416 GB RAM

## SQL 版本

目前可以選擇的 SQL 版本如以下所列：

- MySQL5.6
- 5.7 (預設)
- 8.0PostgreSQL- 9.6
- 10
- 11
- 12 (預設)Microsoft SQL Server- 2017

其實 Cloud SQL 就是預先安裝好各種 SQL 的 instance ， 並且增加了部分客制化的功能。如果使用者有特殊需求，例如特殊版本的 SQL ；或是特殊的硬體規格需求，依然可以透過 Compute Engine 中的 instance 自行架設。
