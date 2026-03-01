---
title: "[Day10] Cloud Spanner (2021 鐵人賽 – Cloud)"
date: 2021-09-25T18:57:00+08:00
draft: false
url: "/2021/09/25/day10-cloud-spanner/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

Spanner 🔧，跟 Cloud SQL 一樣，是一種 RDBMS (Relational Database Management System)，它同時包含了關聯式資料庫與非關聯式資料庫的優點，支援 SQL 的語法，~~然後很貴~~。

Spanner 最大的優點是它可以達成 Horizontal  
Scalable 的資料結構，它可以處理 PB 等級的資料，而且有極高的可靠性(SLA 高達 99.999%)，適合使用於金融與庫存等應用上。

Spanner 是 Google 開發的酷酷資料酷系統，他們也為此發了紙 [Spanner: Google’s Globally-Distributed Database](https://static.googleusercontent.com/media/research.google.com/zh-TW//archive/spanner-osdi2012.pdf)，~~但我是沒有很想看啦QQ。~~

## Cloud Spanner Architecture

Cloud Spanner 會同時將資料放在多個 Zone 中，這些 Zone 可以跨 Region，使用者可以自己設定，想要將資料放在哪個 Region，透過這種概念達到極高的可用性。

多個 Region 間的資料，會透過 Google 的光纜進行同步，並且透過原子鐘來校對時間，確保其原子性。

## Spanner 與 RDBMS / N-RDBMS 比較

![](/uploads/2022/02/f5B9xBn.png)

> 
圖片來源：[Google Cloud](https://cloud.google.com/blog/products/gcp/introducing-cloud-spanner-a-global-database-service-for-mission-critical-applications)
