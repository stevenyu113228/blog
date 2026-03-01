---
title: "[Day6]Virtual Machines (2021 鐵人賽 – Cloud)"
date: 2021-09-21T18:55:00+08:00
draft: false
url: "/2021/09/21/day6virtual-machines/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

今天要來介紹的是雲端的虛擬機器 (VM)，屬於雲端 IaaS 最具有代表的一項產品。相信有許多讀者應該都有在自己電腦上使用虛擬機的經驗吧？例如 Virtual Box 或 VMWare 等程式，虛擬機在實用上有非常多的優點。

## 虛擬機的優點

虛擬機最大的優點就是可以與實體機器隔離，做任何的事情都不會影響到原始的機器。除此之外，在假設 Host 電腦的效能與容量足夠的前提下，我們也可以隨時地變更虛擬機的硬碟大小、CPU大小、硬碟容量等。我們也可以直接針對整台虛擬機的映像(image)檔案進行快照(snapshot)，作為即時的備份使用。

如果我們有很多台的虛擬機，也可以建立虛擬的網路將他們互相連起來；有關雲端的虛擬網路 (VPC)，我們會留到明天再跟各位進行介紹。

上述這些虛擬機的優點，無論是地端與雲端的設備都可以運用的到，而雲端的機器由於是交由服務商進行管理，所以對於正常使用者來說，我們可以視為 host 的機器將近是無限大，想要開多少都沒有問題！

## vCPU 與 Throughput

一個 vCPU 等同於一個硬體的超值行續 (Hyper-Theread)，以目前的 GCP 規定，一台的 VM 最多可以開 96 個 vCPU 。

一個 vCPU 的 網路吞吐量 (throughput) 為 2Gbps。到目前為止，若使用一台大於等於 16顆 vCPU 的裝置，其網路的 throughput 最高為 32Gbps。如果開啟特殊的虛擬機，例如使用到高級 GPU 的 V4 或是 V100 ，則可以到 100 Gbps。

## Storage

虛擬硬碟方面，在 GCP 上面也有幾種選擇，分別是 Standard 、 SSD 與 Local SSD。不同的硬碟選擇，想當然的，就是他們的性能與價格有所差距，大家可以依照自己的需求選擇最適合的硬碟解決方案，而這三種硬碟分別的差距如下：

- Standard可以理解成普通的電腦機械式硬碟
- 透過網路連接上 VM
- 它們的容量可以視需求彈性的增加(只能增加不能減少)
- 最大值在每台 VM 上可以有 257 個 TB
- 最便宜SSD- 相對於 Standard ，可以有更高的 IOPS
- 透過網路連接上 VM
- 也可以在線的增加容量(只能增加不能減少)
- 最大值在每台 VM 上可以有 257 個 TB
- 價格其次Local SSD- 比起 SSD 有更高的吞吐量，且低延遲
- 直接硬體的連接在 VM 上
- 關閉、刪除 VM 後，資料就會被刪除
- 通常拿來作為 swap 使用
- 單顆 Local SSD 最大容量為 375GB
- 一台 VM 最多可以連結 8 顆 Local SSD

## VM Access

通常我們訪問雲端的 VM 都是藉由網路的方式，在大多數情況下， Linux 的作業系統我們會使用預設為 22 Port 的 SSH，而 Windows 作業系統則會使用預設為 3389 Port 的遠端桌面連線 (RDP)。

在 GCP 中，SSH 的連線可以藉由 GCP Console、Cloud Shell 或 Cloud SDK，也可以經由自己習慣的 SSH Client，如 Putty、Moba 等程式。而 RDP 則需要透過 Windows 內建的遠端桌面連線，或是其他 RDP 客戶端。

## 預設的機器類型

雖然我們可以完全的克制化自己的 VM，Google 也有準備了許多預設的，常見需求的機器可以供大家選擇，使用 Google 預設的機器規格，通常會比自己設定便宜許多。預設的分別為：

- Standard標準，提供給最正常的需求使用
- 通常他的 CPU 與 Memory 會是均衡的
- 其 RAM 與 CPU 比值為 3.75 GB : 1 vCPU
- 例如 n1-standard-2 :2 顆 vCPU 搭載 7.5 GB 的 Ram最高可到 96 vCPU , 360 GB RAMHigh-Memory- 提供給需要使用 CPU 較少，而 GPU 較多的使用者
- 其 RAM 與 CPU 比值為 6.5 GB : 1 vCPU
- 例如 n1-highmem-22 顆 vCPU 搭載 13 GB 的 RAM最高可到 96 vCPU , 624 GB RAMHigh-CPU- 提供給需要使用 Memory 較少，而 CPU 較多的使用者
- 其 RAM 與 CPU 比值為 0.9 GB : 1 vCPU
- 例如 n1-highcpu-22 顆 vCPU 搭載 1.8 GB 的 RAM最高可到 96 vCPU , 86.4 GBMemory-optimized- 提供給一些 In-Memory 的資料庫，例如 SAP、 HANA，以及 SQL 資料分析使用
- 其 RAM 與 CPU 比值為 >14 GB : 1 vCPU
- 例如 m1-ultramem-4040 vCPU 搭載 961 GB RAM最高可到 160 vCPU , 3844 GB 的 RAMCompute-optimized- 主要提供單核 CPU 較高的效能 (最高 3.8Ghz)
- 例如 c2-standard-44 顆 vCPU 搭載 16 GB 的 RAM最高可到 60 vCPU 搭載 240 GB RAMShared-core- 會跟他人共用 vCPU，適合效能超小的小程式
- f1-micro : 0.2 vCPU , 0.6 GB RAM提供突發狀況，短期的使用額外的 CPUg1-small : 0.5 vCPU , 1.7 GB RAM

## Preemptible Machine

除了常見的機器之外， GCP 也有提供搶佔式的機器，相較於普通機器而言，它的價格便宜非常非常多，最低可以便宜到 80%。但搶占式機器有可能在任何時候被強制關機，通常搶占式機器會用來做一些 Batch Process 的工作。它最多只能存活 24小時，而在被關機前 30 秒會警告，提供給使用者執行關機腳本。

## Image

GCP 有提供許多不同的 VM Image 供大家使用，包含了各大免費的 Linux 作業系統，以及 Windows 系統。 Windows 系統會依照使用時間收取相關的受權使用費。  
使用者也可以上傳自己的 Custom Image，而此時，對於這個自己的 Image 的安全性，就是使用者須要自己負責的部分了！

## 總結

今天介紹了 GCP 的 VM 相關的一些規格與差別，其實 VM 還有許多內容可以談的，不過礙於時間跟篇幅有限，今天差不多就介紹到這邊。預計明天會與大家介紹 VPC 相關的內容。
