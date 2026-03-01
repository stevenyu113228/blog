---
title: "[Day14] Kubernetes (2021 鐵人賽 – Cloud)"
date: 2021-09-29T19:00:00+08:00
draft: false
url: "/2021/09/29/day14-kubernetes/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

昨天簡介完了 Docker 的容器相關內容，今天我們要來介紹的是 Kubernetes (K8S)，其實 K8S 可以介紹的內容真的非常非常非常多，相信隔壁棚 DevOps 也有許多文章，光是 K8S 就可以介紹完整的 30 天。

Kubernetes，念做(哭ㄅㄋㄟ體死)，它在希臘語中，代表的是舵手，駕駛員的意思；做為 Docker 的管理工具，感覺也有一種管理的味道存在(?)。 Kubernets 是由 Joe Beda、Brendan Burns 和 Craig McLuckie 共同創立，而目前主要的維護是由 Google 的工程師負責。而 Kubernets 又難念，又難寫，連外國人都這麼認為，所以就簡稱為 K8s，8是因為 `ubernete` 剛好有 8 個字。

## Kubernetes

Kubernetes 可以是為一個高層次的 Docker 抽象化管理器，在最高層級中，Kubernetes 可以透過一系列的 API 對於 Cluster 中的 Node 、 Container 進行管理。

在主從式架構中，我們可以將一個 Cluster 中的內容歸類為兩種元件，Master 與 Node， Master 負責進行控制與管理 Nodes；而 Nodes 則負責執行 Container 的內容。在 Google Cloud 中，每一個 Node 代表了一台 Compute Engine 的 Instances。

## GKE

Google Kubernetes Engine 簡稱 GKE，是由 Google 管理的的 K8s 引擎，支援 Google Cloud 上各種不同規格的 Compute Engine instances。

透過 Gcloud 的指令 `gcloud container clusters create k1` 我們就可以快速的建立一個名為 `k1` 的 Cluster。

## Pod

在每一個 Node 中，我們也可以擁有多個不同的 Pods。Pod 是 Kubernets 中的最小單位，在一個 Pod 中，我們可以擁有一個或多個 Container ，每一個 Pod 提供了一組 IP 以及 Port，連接到 Pod 中的 Container 中；在同一個 Pod 中的 Container 可以透過 localhost 以及固定的 port 進行互相的溝通。

我們可以透過 `kubectl` 的指令 `kubectl get pods` 快速的觀察與檢視每個 Pods 中的原件。

## Load Balancer

在預設情況下，每一個 Pod 中的資料僅能在 GKE 的 Cluster 內進行存取，如果我們希望將服務公開到外網，則可以使用 `kubectl expose` 的指令，將 nodes 接上附載平衡器 (Network Load Balancer)，並透過外網 IP 連接進來。

## Auto Scaling

Auto Scaling 可以算是 K8s 中，超級重要的一個功能，假設我們在 Container 中放置了一個 Web Server ，如 Apache 或 Nginx。已知我們的服務可能在尖峰時間會有非常多人使用，而離峰時間則少很多，我們就可以透過 Auto Scaling 的方式，依照 CPU 使用率等規則，自動化的縮放我們的 node ， 當 CPU 使用率過高時，我們就開啟更多的 node ，以因應大量的請求。

關於 autoscale ，可以使用 `kubectl autoscale` 指令來達成。
