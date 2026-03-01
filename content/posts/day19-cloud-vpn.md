---
title: "[Day19] Cloud VPN (2021 鐵人賽 – Cloud)"
date: 2021-10-04T19:04:00+08:00
draft: false
url: "/2021/10/04/day19-cloud-vpn/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

## Cloud VPN

Cloud VPN 可以將自己本地的網路直接的連接到 GCP 的 VPC 網路之中，透過 IPSec 協議進行加密，可以保護資料在網路上傳輸的安全性。

Cloud VPN 屬於由 Google 託管的服務，提供了 99.9% 的 SLA，並支援 Side to Site 的 VPN、靜態路由、動態路由、IKEv1 與 IKEv2 的密碼等功能。

在建立 Cloud VPN 前，我們需要建立本地端的 VPN Gateway 、 雲端的 VPN Gateway 以及兩個 VPN Tunnel， Cloud VPN 的 Gateway 屬於一個 Regional 的資源，因此需要使用一個 external IP 。

本地端的 VPN Gateway 可以是硬體的主機，也可以是軟體的 VPN 服務，安裝在本地或其他雲端伺服器中。

目前 Cloud VPN 的最大傳輸單位 (MTU) 為 1460 Bytes。

## Cloud Router

Cloud VPN 支援靜態以及動態路由，如果需要使用動態路由，則需要開啟 Cloud Routers 的功能。 Cloud Routers 支援透過 BGP (Border Gateway Protocol) 管理 Cloud VPN Tunnel 的路由。

要設定 BGP ，必須為兩端的 VPN 分配一個額外的 IP Address，這兩個 IP Address 必須是 link-local IP，於 IP Range 169.254.0.0/16 之中。這個 IP 位置不屬於兩邊網路的 IP，而是專門提供給 BGP 使用。
