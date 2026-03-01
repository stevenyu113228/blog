---
title: "[Day17] Cloud Run (2021 鐵人賽 – Cloud)"
date: 2021-10-02T19:03:00+08:00
draft: false
url: "/2021/10/02/day17-cloud-run/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

Cloud Run 是這次要介紹的最後一種部屬服務方式。它是一種基於 Container 的 Serverless 服務。比起 Google Kubernetes Engine，使用者不需要自己的管理 K8s 的服務。

Cloud Run 背後使用了開源的 Knative 服務，使用者可以直接的使用 Cloud Run 管理自己的 Container。除此之外，它最大的特色是，使用者也可以將 Cloud Run 的服務，搬回自家的 Kubernetes Engine 上進行執行，連接上自己的 VPC。

在使用 Cloud Run 之前，使用者必須先將自己的 Container 上傳至 Cloud Container Registry 中。

Cloud Run 與 Kubernetes Engine 都可以結合一整套的 Google 服務的生態系，達到 DevOps 的功能。舉例來說，我們最終的目標是要部屬一個由 Python 的服務，將程式寫完後，我們可以將程式推送到 Cloud Source Repositories 中，並串接 Cloud Build 的 Trigger，將 Docker 給 Build 起來後，放置於 Container Registry 中，再提供 Cloud Run 進行執行。

## Cloud Application Deployment 比較

這幾天，我們總共介紹了 5 種不同，可以部屬自己程式的服務，分別是 Compute Engine、Kubernetes Engine 、 Cloud Run 、 Cloud Functions、 App Engine 。

- Compute Engine需要管理最底層的作業系統跟硬體
- 完全由自己掌握整台機器Kubernetes- 需要透過容器化進行管理
- 且需要自己管理 K8s 的 ClusterCloud Run- 需要透過容器化進行管理
- 但不想自己管理 K8s 的 ClusterCloud Functions- 不需要容器化
- 基於事件觸發App Engine- 不需要容器化
- 不用基於事件觸發
- 主要設計給微服務 (Microservices)
