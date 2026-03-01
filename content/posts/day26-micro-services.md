---
title: "[Day26] Micro Services (2021 鐵人賽 – Cloud)"
date: 2021-10-11T19:08:00+08:00
draft: false
url: "/2021/10/11/day26-micro-services/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

在前幾天的 App Engine 與 K8S 中，或許我已經大致的提過 Micro Services ，今天來試著更詳細的介紹 Micro Services 的概念。

Micro Services 不限於任何的程式語言，無論 Python、Node.JS 或 Go 都有辦法實現，它最大的特色就是「模組化」，把一個大程式給切割成一塊一塊的模組，方便維護與管理，當面對一個非常龐大的系統時， Micro Services 也可以快速的定位發生的問題。

與 Micro Services 相對的是 Monolithic application 單體式應用程式，也就是將所有的功能全部寫在一起，這種方式以連接角度而言，設計會比較單純與簡單，開發時程上也會比較快。但當整個系統日益增大，則維護上會較為困難。

相較於單體式程式，微服務還有另外一個特色是可以快速的將任何一個模組進行擴展，假設我們有一個銷售網站，而微服務切分了物品的展示前端，與實際購買，處理訂單的後端。在雙 11 等活動時，假設我們已知大多數的客戶已經將需購買的商品加至購物車，這種情形下，我們就可以單純的透過 K8S 的 Scaling 相關設定，設定擴展銷售後端的模組，而不需要連同前端一起進行擴展，減少租用機器的費用。

微服務最大的特點就是每個服務應該獨立自主，因此服務間最好需要有獨立的資料庫，服務間唯一的溝通方式是透過 API (最常見的是透過 RESTful) 進行交互。

## 12 Factor App Design

基於 Micro Services 設計的 SaaS 最佳實踐方式，目前最有名的是 The 12-Factor App ，關於相關的介紹可以參考去年 [Miles 大大的鐵人賽文章](https://ithelp.ithome.com.tw/articles/10253303)。簡單來說，我們的開發過程可以遵守 12 Factor 的方式，有效的開發方便移植、擴展的程式。而這 12 Factor 依序是：

1. Codebase- 使用版本管理系統，例如 Git。
2. Dependencies- 使用套件管理系統，例如 pip、npm。
3. Config- 不要將 API Key 等機密資料放至於程式碼字串中。
4. Backing services- 資料庫，快取等資源可藉由 URL API 訪問。
5. Build, release, run- 嚴格分離程式碼的 Build 與 Run 階段。
6. Processes- 一個程式可分為一個或多個 Process，每個 Process 應為 stateless。
7. Port binding- 透過 Port binding 方式讓程式可以聯外。
8. Concurrency- 透過 Process Model 進行水平擴展。
9. Disposability- 快速啟動，追求穩定性，並可以優雅的關機。
10. Dev/prod parity- 保持開發與部屬的環境一致。
11. Logs- 將 Log 透過 event Stream 輸出。
12. Admin processes- 將維護與管理視為一次性的任務。
