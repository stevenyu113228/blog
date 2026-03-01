---
title: "[Day28] Security (2021 鐵人賽 – Cloud)"
date: 2021-10-13T19:09:00+08:00
draft: false
url: "/2021/10/13/day28-security/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

在網路世界中，安全永遠是最最重要的事情，而雲端安全當然也不例外。任何的安全問題都來自於人為的疏忽，部分雲端上面會遇到的安全問題，在地端的機器上也會遇到，例如常見的 OWASP TOP 10 上的網路攻擊，或是 DDoS 等。除了這種面對駭客的攻擊之外，我們也要注意公司內的意外~~內賊~~部分，如[這則新聞](https://news.ltn.com.tw/news/society/breakingnews/2012042)，三立電視的離職員工透過 Tor 返回原公司的 AWS 帳號中刪除檔案。

今天我主要會介紹的是 Google Cloud 的管理層面，有哪些可以注意的地方，這邊會介紹人、機器與網路三個層面。

## 人

人的問題為例，最最重要的就是 IAM 相關的管理，身為一個雲端的管理者需要使用資料夾的方式管理公司內的各個部分，並且掌握「保持最低權限」的原則，盡量的讓使用者在可以順利做事情的最低權限，特別注意 Owner 與 Editor 的權限不能無止盡的發。而當公司人員掉離部門或離開公司後，應該要立刻修改它的 IAM Rule。

## 機器

機器相關的問題，一樣回到 IAM 上，我們需要確保每台機器的 Services Account 權限是否合宜，是否有可能使用者或惡意的攻擊者從低權限的機器，藉由 Services Account 的 Rule 而橫向移動到其他抬高權限的機器之中。如果是自己管理的作業系統，需要確認作業系統是否有安全性的更新需要進行修補。而部屬的程式上的安全則暫時不在今天雲端的討論範圍內，可以參考我隔壁棚的滲透測試文章。

## 網路

網路層面的安全其實一樣基於「最低權限」的原則，一台不需要聯外的伺服器就不要給它 Public 的 IP，而需要對外的服務也需要透過防火牆來限制 Port Forwarding；可以透過 Google Cloud 的 Load Balancing 進行 DDoS 的檢測，並透過 Cloud CDN 或 Cloudflare 等 CDN 服務進行防禦。Google Cloud 也有提供 Cloud Armor 與 WAF 等服務保障網路層面的安全。
