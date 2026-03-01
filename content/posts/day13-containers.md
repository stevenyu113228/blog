---
title: "[Day13] Containers (2021 鐵人賽 – Cloud)"
date: 2021-09-28T19:00:00+08:00
draft: false
url: "/2021/09/28/day13-containers/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

我們都知道，Compute Engine 屬於 IaaS ， 而 App Engine 等服務屬於 PaaS，在 PaaS 底下，使用者大多不需要處理系統底層的問題。

但有些時候，我們會希望可以同時擁有 IaaS 與 PaaS 的優點，可以設定部分底層的資源，又希望可以透過雲端平台幫我們自動化的管理，這個時候，就可以使用 Kubernetes (K8S) 的服務，關於 K8S 相關的內容預計將於明天介紹，今天會介紹 K8S 底層的東東： Container。

虛擬化容器與 VM (虛擬機) 非常類似，但是原理上有所差異。通常， VM 會模擬整個電腦硬體、CPU，包含指令集等所有的內容，因此 VM 最被大家詬病的問題就是，需要浪費非常多的系統資源。在 CPU 層面，需要模擬整個電腦硬體，而在儲存層面，也需要安裝整套的作業系統。

容器的虛擬化則是將作業系統層進行虛擬化，與當前在運行的作業系統共用資源，省去了虛擬化硬體的部分，可以達到非常的輕量化。我們可以將自己的服務打包在容器中，未來在搬移時，僅需要將容器移植到其他台電腦，即可快速的部屬各種不同的服務。

## Docker

雖然說，Container 的技術不僅只於 Docker，但目前來說，Docker 算是最知名，最通用的一款 Container 技術。

以 Docker 為例，如果我們需要建立一個 Docker 的容器，需要一個 Dockerfile ，定義我們容器中各種參數，例如需要繼承哪一套的 image，並且需要在自己的容器中執行哪些的參數；掛載的檔案；網路連線等。

每一個容器中，可以安裝多個程式，例如 Apache 、 MySQL 、 Python 等，不過根據 Docker 官方的建議，最好每一個容器僅安裝一套的服務，可以稱為 (Micorservices) 微服務。

我們可以在地端，自己的電腦中建立好一套完整的服務，並將 Docker 的各種服務部屬到 Compute Engine 的 Instances 上。

當我們有了一系列的 Docker Container，接下來需要面臨的挑戰就是，容器的管理了，那麼，關於容器的管理，我們明天再繼續介紹！
