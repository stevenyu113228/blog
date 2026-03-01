---
title: "[Day11] Big Table (2021 鐵人賽 – Cloud)"
date: 2021-09-26T18:58:00+08:00
draft: false
url: "/2021/09/26/day11-big-table/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

~~雲端大桌子~~，是一種 NoSQL 的資料庫，可以支援 TB 等級的應用，且支援 HBase 的 API，可使用於大數據、 Hadoop 等環境之中。

## NoSQL

講到 NoSQL 就要回來重新提一下什麼是 SQL ， SQL 就像一張~~桌子~~表格，我們每次的新增都會依照表格的固定 Column 來新增 Row，每一條的 Row 都要依照著 Column (Table schema )的欄位來填寫。

很多情況下，我們新增的資料不見得每次都會把每個 Column 給填滿，這種情形下就會成為稀疏矩陣的感覺，造成資源的浪費。而 NoSQL 正是基於這種想法，提出的非結構化資料儲存系統。

## 適用環境

Cloud Bigtable 擅長處理批次的 MapReduce 等操作，以及 Stream 的處理/分析，以及機器學習等。在實際的場合底下，Bigtable 適合以下幾種資料

- 銷售資料例如購買紀錄
- 使用者偏好金融資料- 交易紀錄
- 股票價格
- 貨幣匯率物聯網資料- 各種感測器資料
- 記錄檔案
- 分析報告時間序列資料- 例如 CPU 以及 Memory 的使用率等

## Bigtable 的最大特色

- 特色大 ： 資料可以支援超過 1 TB
- 快 ： 高吞吐量
- NoSQL ： 非關聯資料庫適用資料- 時間序列
- 大數據
- 機器學習
