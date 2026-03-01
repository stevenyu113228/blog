---
title: "[Day7]Virtual Networks (2021 鐵人賽 – Cloud)"
date: 2021-09-22T18:55:00+08:00
draft: false
url: "/2021/09/22/day7virtual-networks/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

今天來介紹虛擬網路的部分，既然我們都有各種的虛擬機器、虛擬服務，那想當然，這些服務就是透過虛擬網路進行連線。虛擬網路如同現實網路一樣，也有許多需要在意與設定的地方，例如 IP 、 Routing 或防火牆等。

## VPC

虛擬私人網路 (VPC, Virtual Private Cloud) ， 就如同現實生活中的 LAN ， 可以將虛擬網路彼此的進行連接或隔離。 一個 Project 預設可以有 5 個虛擬網路，這些網路除了自己使用之外，也可以跟其他的 Project 互相共享。這些虛擬網路中的 IP 沒有嚴格的限制範圍；我們也可以設定自己在歐洲與亞洲的機器於同一個區網內。我們也可以藉由 Cloud VPN 等方式將自己地端的設備接入雲端，Cloud VPN 相關技術將於之後的文章再跟大家介紹。

VPC 在 Google Cloud 中有三種模式，分別是 Default 、 Auto 與 Custom

### Default Mode

在每一個 Project 中都會預設一個 Default Mode 的網路，他們在每一個 Region (可跨 Zone) 中都是一個 Subnet，所以，它有預設的防火牆 Rule。事實上， Default Mode 的 VPC 網路，就是透過接下來要介紹的 Auto Mode 生出來的。

### Auto Mode

Auto Mode 如同 Default Mode，它會自動生成，每一個 Region 中會有一個 Subnet，而每一個 Subnet 預設的 IP Range 使用了 /20 的子網路遮罩，最高可以擴充為 /16。

### Custom Mode

Custom Mode 不會自動的生成任何的子網路，可以完全的自由設定子網路，只要 IP 位置不要互相重疊即可。這邊設定的 IP 位置需要符合 RFC 1918。也就是下列三大區段

- 24 bit 區段`10.0.0.0 – 10.255.255.255`
- `10.0.0.0/8`20 bit 區段- `172.16.0.0 – 172.31.255.255`
- `172.16.0.0/12`16 bit 區段- `192.168.0.0 – 192.168.255.255`
- `192.168.0.0/16`

我們可以自由地將任何的 Mode 轉換為 Custom Mode 以設定更多自己的需求，但我們不能將 Custom Mode 重新的轉換為 Auto Mode。

## IP Address

在 GCP 中，每個 VM 可以有兩個 IP， 其中一個是 Internal IP ， 而另外一個則是 External IP。 Internal IP 是依照 Subnet 透過 DHCP ， 每 24 小時自動更新一次的，且 VM 名稱與 IP 會被註冊在 Network-scoped DNS 之中； External IP 則有兩種，固定式 IP 以及浮動式 IP，不過 VM 不會知道自己的 External IP 是多少，它們是透過 Internal IP 進行自動 mapping 的。

Network-scoped DNS 對應的 Doamin，其 FQDN 為 `[hostname].[zone].c.[project-id].internal`，會藉由 internal DNS resolver (`169.254.169.254`)進行處理。

## Firewall

VPC 的連線會經過防火牆，而防火牆的規則會套用到整的網路中，在預設狀態下，防火牆的規則為：禁止所有的入口(Ingress)，以及允許所有的出口(Egress)。我們可以依照指定的規則，對輸入輸出的 IP 位置、 Protocol 以及 Port 進行設定，設定為 Allow 或 Deny。也可以藉由 VM 的 Network Tag 對於特定的 VM 及群組進行設定。
