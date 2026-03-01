---
title: "[Day15] App Engine (2021 鐵人賽 – Cloud)"
date: 2021-09-30T19:01:00+08:00
draft: false
url: "/2021/09/30/day15-app-engine/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

今天要來介紹 Google Cloud 中的 App Engine，App Engine 是一款 PaaS 的服務，可以超簡單的進行部屬與維護，使用者也可以專注於開發程式，而不需要煩惱任何關於底層架構的問題。 PaaS 適合用於可擴展的網路應用程式、行動服務的後端等，例如 Chatbot 、輕量化的 API 等服務都很適合可以部屬在 App Engine 上。

App Engine 提供了許多內建的 API，例如 NoSQL 資料存儲、記憶體快取、負載平衡、應用程式 Log 紀錄、身分驗證 API 等。藉由 Security Scanner 的功能， App Engine 可以自動化的掃描與偵測程式的漏洞，提醒開發者進行修補。

App Engine 會透過使用者的流量，自動化的擴展程式後端的伺服器，使用者只需要將程式碼上傳至 App Engine 的管理後臺即可，剩下的問題全部都由 Google Cloud 進行處理。目前，常見的開發環境例如 Eclipse、IntelliJ、Maven、Git、Jenkins、PyCharm 等程式都支援部屬 App Engine 的服務。

App Engine 目前有兩種環境，分別為 Standard 與 Flexible。

## Standard Environment

標準的環境可以簡單的部屬使用者的 APP，支援 Autoscale 等功能，相較於 Flexible 便宜不少。它有每天的免費額度可以使用。在標準的環境中，有提供幾個常見的 runtime 供大家使用，包含了 Java、Python、Go 與 PHP。

在使用 App Engine 時，必須符合一定的 Sandbox 規範，例如不可將檔案寫到本地的硬碟中，需要透過資料庫的形式儲存資料；每一個 Requests 最長的 Timeout 為 60 秒；對於安裝第三方的程式、Library 也有限制。因此需要基於 App Engine 的 SDK 進行開發。

使用者可以在本地透過 App Engine 的 [SDK](https://cloud.google.com/appengine/downloads) 進行開發，模擬 Sandbox 的環境，並透過 SDK 將服務上傳、部屬到雲端中。

## Flexible Environment

Flexible 的環境相較於標準的環境，有更高的靈活度，不需要受限於 Sandbox 的各種約束。可以藉由上傳 Dockerfile 部屬自己的 Container 內容，可以將資料寫入硬碟中、安裝第三方軟體，更可以透過 SSH 連線至自己的 Container 內。相較於 Standard 的環境，Flexible 的環境價格較高。

## 比較

- Kubernetes Engine屬於 Hybrid 的 Service Model，介於 IaaS 與 PaaS 之間
- 需要自行管理 ClusterApp Engine Flexible- 屬於 PaaS
- 由雲端服務商負責管理機器
- 靈活度介於 K8s 與 App Engine Standard 之間App Engine Standard- 屬於 PaaS ，較 Flexible 更高了一層
- 限制指定的程式語言 (Java, Python, Go, PHP)
