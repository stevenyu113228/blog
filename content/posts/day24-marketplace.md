---
title: "[Day24] Marketplace (2021 鐵人賽 – Cloud)"
date: 2021-10-09T19:07:00+08:00
draft: false
url: "/2021/10/09/day24-marketplace/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

對於初學者來說，自動化的部屬真的是一件非常累人的事情，從雲端架構開始就有一大堆東西要學，再加上昨天提到的 IaC，如果真的要講應該也可以寫成 30 天分的教材QQ。如果我們只是一個非常非常淺的初學者，有沒有什麼方法可以快速的自動化部屬服務到 GCP 中呢？當然有的，這就是 Marketplace。

我們可以很簡單的把 Marketplace 想像成 App Store 或 Play 商店，我們想要的服務，有很大的機會都不需要自己重造輪子。舉例來說，我們假設需要架設一個 WordPress CMS 的 Blog，如果透過 Compute Engine，可能會需要先設定系統，安裝 PHP 、 MySQL，再來安裝 WordPress。而如果希望資料庫分離的話，雖然可以透過 Cloud SQL 來處理 MySQL 的部分，不過依然需要處理很多底層的東西QQ。

在 Marketplace 之中，我們可以一鍵的部屬別人設定好的安裝檔，大多數都是透過 Deploy Manager 建置的完整腳本，如上述的 Wordpress，我們可以快速的依照它人設定好的腳本來進行部屬，常見的 MongoDB、ELK 等資料庫也都可以快速的在 Marketplace 中進行部屬。

Marketplace 中，也有包含了 SaaS 等相關服務的方案可以選擇，部分的方案是需要付費的，使用與購買前也需要特別注意哦！
