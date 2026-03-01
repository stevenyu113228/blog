---
title: "[Day29] Cost Plaining (2021 鐵人賽 – Cloud)"
date: 2021-10-14T19:10:00+08:00
draft: false
url: "/2021/10/14/day29-cost-plaining/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

終於到了倒數第二篇文章了，今天來介紹雲端付費相關的計畫，透過一些小技巧，可以減少在雲端上的很多開銷哦！

首先要介紹的是 [Google Cloud Pricing Calculator](https://cloud.google.com/products/calculator/)，透過計算機，我們可以快速的估計機器的相關成本。

![](/uploads/2022/02/D0cTQLN.png)

Compute Instances 的 VM，在啟動一段時間後，如果 CPU 使用率或 Memory 使用率一直很低，Google Cloud Console 上也會有建議，可以依照建議修改自己的機器配置。VM 的計價方式也有一些優惠，如果 30 天內持續租用，可以獲得較多的優惠！而如果是批次性運算的程式，沒有非常大的即時性需求，也可以試著採用 Preemptible VM instances。

儲存相關的話，VM 的硬碟是可以在開機狀態下進行擴展的，所以不需要一開始就開的非常大，可以減少不少的開銷。也可以比較標準硬碟與兩種 SSD 的方案，是否真的有這些需求。Cloud Storage 若要存放長期，極少存取的備份檔案，可以優先的採用 Nearline 或 Coldline 、 Archive 等方式。

我們也可以結合 Cloud 的 Monitoring 相關服務隨時監控各種服務的狀態，如有閒置或不合適的情形發生，就即時的進行處理。
