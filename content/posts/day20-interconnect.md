---
title: "[Day20] Interconnect (2021 鐵人賽 – Cloud)"
date: 2021-10-05T19:05:00+08:00
draft: false
url: "/2021/10/05/day20-interconnect/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

今天來與各位介紹雲端的 互連網路(Interconnect) 與 對等互聯(Peering) 相關服務。Interconnect 與 Peering 最大的差別在於，Peering 屬於 OSI 第 3 層的連接，而 Interconnect 屬於 OSI 第 2 層的連接。而他們又再分為了 Direct Peering、Carrier Peering、Dedicated Interconnect、Partner Interconnect。

## 對等互連 Peering

直接對等互連 (Direct Peering) 可以在自己的連結上透過 BGP 來切換與 Google 的連線，可以讓自己的企業區域網路直接的連接上 Google 的邊際網路。

電信業者互連 (Carrier Peering)，透過電信業者作為中介，進行連接。

## 互連網路

專屬互聯 (Dedicated Interconnect)，提供了直接連線到 Google 的專用網路。可以選擇 10 GBps 或 100 GBps 的專線連接上 Google。也可以使用支援 RFC1918 的 IP Address 進行連線。

合作夥伴互聯 (Partner Interconnect)，如同電信業者互連，可以藉由電信業者、Google的合作夥伴進行連線。

## 比較

關於這些連接方式的詳細比較，可以參考下圖：

![](/uploads/2022/02/TLpykPL.png)

(圖片來源：Connecting to Google Cloud: your networking options explained https://cloud.google.com/blog/products/networking/google-cloud-network-connectivity-options-explained)
