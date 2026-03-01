---
title: "[Day21] Load Balancer (2021 鐵人賽 – Cloud)"
date: 2021-10-06T19:06:00+08:00
draft: false
url: "/2021/10/06/day21-load-balancer/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

Load Balancer (負載平衡器) 與 Auto Scaling 都算是雲端設備中非常重要的一部分，負載平衡器可以將流量分散到多台不同的機器，而這些服務對外只需要一個 IP 即可。透過 Load Balancer ， 我們可以增加資料的可用性與穩定性，並且可以透過 Auto Scaling 的方式自動增減在 Load Balancer 後的伺服器數量。

在 GCP 中， Load Balancer 被分為兩種，Global 與 Regional。Global 的 Load Balancer 有 HTTP(S)、SSL Proxy 與 TCP Proxy，在全球的使用者可以使用相同的 IP ，存取到就近國家的主機，減少傳輸的延遲；Regional 的 Load Balancer 則有 Internal 與 Network Load Balancer，Internal Load Balancer 基於 Google 開發的 SDN [Andromeda](https://www.usenix.org/system/files/conference/nsdi18/nsdi18-dalton.pdf)進行實作，而 Network Load Balancer 則使用 [Maglev](https://static.googleusercontent.com/media/research.google.com/zh-TW//pubs/archive/44824.pdf) 的分散式系統進行實作。

## Instance Template / Group

在開始講 Load Balancer 之前，我們需要先了解 Instances Group，在假設我們 Load Balancer 背後每一台機器都是一樣的前提下，我們會需要建立很多台內容完全一樣的 Compute Engine VM Instances，這種狀況下，我們就可以使用 Compute Engine 中的 Instance Template，作為 Load Balancer 未來開啟的模板。Instance Template 就像是一個 VM 的模板，他的建立方式與建立 Instances 將近完全一樣，設定 VM 的 Image、Startup Script 等。

當我們建立好 Instance Template 後，接下來需要建立 Instance Group，在這個時候我們就可以開始設定機器的細節與 Auto scaling 的規則。例如假設這個 Instance 是 Web Server，則我們需要開啟 Health Check 來對網頁進行監控，我們也可以設定這個 Group 最多可以開多少台機器等資訊。

在這邊，我們也可以對規則進行設定，例如針對 CPU 的使用率、 Load Balancing 的容量、各種監控的指標等。

## HTTP(S) Load Balancing

HTTP 與 HTTPS 的 Load Balancing 屬於全球性的 Load Balancing，HTTP 針對了 80 與 8080 Port，而 HTTPS 則針對 443 Port，同時支援 IPv4 與 IPv6。

使用者也可以藉由 URL map 來針對，將指定的 URL 資訊導到指定的伺服器中，可以讓伺服器的工作更單一化，更方便進行管理與維護。例如 `http://example.com/dog` 與 `http://example.com/cat` 導到不同台的機器。

Load Balancing 背後的機器就是由 Instance Group 組成的 Backend Service，可以藉由 Health Cheak 的規則自動的對機器進行縮放。

## SSL / TCP Proxy Load Balancing

SSL Proxy 透過加密的方式進行傳輸，主要針對各種非 HTTP 的加密封包，它可以自動化的管理憑證，也提供了安全性的更新，以及設定 SSL 的政策等功能。以憑證的角度來看，我們只需要將憑證放在一個地方，不需要重新的部屬在多台機器上面。

TCP Proxy 則與 SSL Proxy 相似，一樣是針對非 HTTP 的 TCP 封包進行 Load Balancing，而 Proxy 與 Backend 的連線可以選擇使用 TCP 與 SSL，使用 SSL 會相對比較安全。

## Network Load Balancing

Network Load Balancing 是 Regional 的服務，它屬於非 Proxy 形式的 Load Balancing，支援 UDP 、TCP / SSL 的封包，而後端則支援 Instances Group 與 Target Pool。

Target Pool 定義了一群 Instances，負責針對 Forwarding rule 進行接收，而這些 Instances 必須在同一個 Region 中，每一個 Target Pool 只能有一個 Health Check。

## Internal Load Balancing

Internal Load Balancing 支援 TCP 以及 UDP 的流量，它透過 VPC 的 Private IP 進行流量的傳遞，這種形式的 Load Balancing 是完全提供給 Internal 使用的，因此也不需要有外網的 IP。透過這種方式設定 Internal 的 Load Balancing ，可以讓所有的網路流量都在 Google 的網路內，降低延遲。
