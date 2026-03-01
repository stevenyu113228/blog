---
title: "[Day4] Google Cloud (2021 鐵人賽 – Cloud)"
date: 2021-09-19T18:52:00+08:00
draft: false
url: "/2021/09/19/day4-google-cloud/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

今天的內容會跟各位介紹 Google Cloud 相關的基礎知識，希望不會不小心的講成像業配文QQ，先聲明，我沒有拿到 Google 給我的任何 $$ QQ。

## GCP 提供了多種類型的服務

GCP 的服務，從低層級到高層級，包含了 IaaS 、 Hybrid 、 PaaS 、 Serverless logic 與 Automated elastic resources。越低層級的內容，使用者可以控制的參數就越多，設定也比較複雜；越高層級的服務須要設定的參數就越少，可以快速且方便的使用，不過相對應的，彈性與客制化程度就比較低。特別是機器學習的部分，Google 提供了許多方便的 API 供大家使用，這些 API 甚至可以讓不會寫程式的人快速的接入各種方便的應用。

以下為 GCP 常見的服務列表

- 運算資源Compute Engine
- Kubernetes Engine
- App Engine
- Cloud Functions
- ……儲存資源- Bigtable
- Cloud Storage
- Cloud SQL
- Cloud Spanner
- Cloud Datastore
- ……大數據- BigQuery
- Pub/Sub
- Dataflow
- Dataproc
- Datalab
- ……機器學習- Natural Language API
- Vision API
- Machine Learning
- Speech API
- Translate API
- ……

## Google Network

根據資料顯示， Google 目前在全世界的網路下，成載了世界的 40% 的流量，是世界上最大的網路商。它的海底光纜的總長度更超過了 10 萬公里，投資了數十億美元建至，確保了 Google 各種服務的穩定性。

![](/uploads/2022/02/69db0-edgepoint_2x.png)

> 
圖片來源 : [Google Cloud 據點](https://cloud.google.com/about/locations#regions)

## Region , Zone

Google 把各個資料中心的區域，區分成了 Region 與 Zone。 Region 是由多個 Zone 組成的實體地理位置，通常在同一個 Region 中，它們的網路傳輸延遲會低於 5 ms。

Zone 是 Region 中，可以部屬裝置的區域，每一個 Zone 可以視為一個故障域(failure domain)，在考慮可用性的前題下，我們可以把服務部屬在同個 Region 的多個 Zone 中，以防止故障的發生。而通常選擇不同的 Region，是為了可以讓服務更接近使用者，以降低延遲。

舉例來說，Google 在台灣彰濱工業區的資料中心，其 Region 為 `asia-east-1` ，它有 3 個 Zone ，分別是 `asia-east1-a`、`asia-east1-b`、`asia-east1-c`。

GCP 的服務資源，也有分為 Zonal 、 Regional 與 Multi-regional 。Zonal 的資源就只會存在於一個 Zone 中，如果 Zone 發生了故障，則資源就完全無法存取。 Regional 的資源會在 Region 中有更高的可用性。而 Multi-regional 則同時將資源跨在多個 Region 中，有更高的可用性、性能與效率。

![](/uploads/2022/02/51f41-regions_2x.png)

> 
圖片來源 : [Google Cloud 據點](https://cloud.google.com/about/locations#regions)
