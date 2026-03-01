---
title: "[Day18] Operations Suite (2021 鐵人賽 – Cloud)"
date: 2021-10-03T19:04:00+08:00
draft: false
url: "/2021/10/03/day18-operations-suite/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

今天我們來介紹雲端的監控， Cloud Operations Suite 。這個服務以前被稱做 Stackdriver 。雲端的監控總共可以分為，Monitoring、Logging、Error Reporting、Tracing 以及 Debugging。

這些監控相關的程式 (Agent) 支援跨平台，除了 GCP 之外，也可以[安裝](https://cloud.google.com/monitoring/agent/ops-agent/install-index)在 AWS 或是自己的主機上面，這些服務皆有免費額度可以使用。

## Monitoring

伺服器的監控是一件非常重要的事情，這是 SRE (Site Reliability Engineering) 最重要的工作。

Monitoring 支援了各種的指標，包含平台、系統、應用程式，可以將圖表、警報顯示於儀表板 (Dashboards) 中。

![](/uploads/2022/02/8Burgrc-1024x360.png)

## Logging

Cloud Logging 提供分析 GCP 以及 AWS 上的各種事件，Logging 包含了 紀錄的儲存、使用者介面 (Log Viewer) 以及 API，可以透過程式化的方式管理 Log。

與 Monitoring 一樣，我們也可以透過[腳本](https://dl.google.com/cloudagents/install-logging-agent.sh)將 Agent 安裝在自己的伺服器中，監控伺服器的網路流量、IP 來源等資訊。如果我們想要 Query 大量的 Logging 資料，也可以使用 BigQuery 等程式來達成。

## Error Reporting

Error Reporting 可以統計、分析在雲端服務中的各種錯誤，提供了錯誤的通知、儀表板等功能。支援 App Engine 、 Apps Script 、 Compute Engine 、 Cloud Functions 、 Cloud Run 、 GKE 、 Amazon EC2 等平台。當我們使用以上的平台，部屬過程與運行過程發生任何的錯誤，都可以在 Error Reporting 中找到相應的 Log。

目前支援的程式語言有 Go、Java、Node.js、PHP、Python、Runy。

## Tracing

Cloud Tracing 是一個分散式的追蹤系統，可以追蹤並蒐集延遲 (latency) 相關的資料。蒐集的目標包含App Engine 、 HTTP(S) 附載平衡器，或其他 Cloud Trace SDKs 支援的目標。可以接近即時的顯示資訊，包含了延遲報告、每個 URL 相關的延遲資訊。

## Debugging

Cloud Debugging 可以在不停止程式的狀況下檢查程式。包含了 Snapshot 與 Logpoints 功能。

Snapshot 功能可以將程式的 Call Stack 以及區域變數等資料全部都 Dump 出來，以方便偵錯；Logpoints則可以在服務中插入 Log，方便監控。

目前支援的語言有 Java、Python、Go、Node.js、Ruby、PHP 以及 .NET Core
