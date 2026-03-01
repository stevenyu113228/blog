---
title: "[Day22] Deployment Manager (2021 鐵人賽 – Cloud)"
date: 2021-10-07T19:06:00+08:00
draft: false
url: "/2021/10/07/day22-deployment-manager/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

從今天開始的內容，會相對比較簡單一點點，我們來介紹一些自動化的部屬方式。假設說，我們設定好了一系列的 Instance 、 VPC 架構 、 防火牆規則等相關的服務，未來一個方案需要重新利用時，就可以透過 Deploy Manager 與明後天會介紹到的 IaC 等方式進行快速的部屬。

我們部屬 Google Cloud 的服務，通常會使用 Cloud Console、Cloud Shell 或 SDK，在還沒有學習到 Deployment Manager 前，如果我有自動化需要部屬大量的設備，通常我會直接把需要部屬的內容寫成 Bash 的 Shell Script ，並透過 Cloud Shell 執行，這種方式很暴力，但很有用 XDD。

Deployment Manager 屬於 Infrastructure Automation Tool，可以透過 GUI 的介面，在 Cloud Console 對於不同的資源在部屬前進行規劃，也可以將各種資源們建立成模板，方便重複使用。Deploy Manager 背後使用了各種 GCP 底層的 API 進行設計，基本上可以支援所有 GCP 的服務部屬。
