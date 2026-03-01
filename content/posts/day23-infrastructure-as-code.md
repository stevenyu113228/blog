---
title: "[Day23] Infrastructure as code (2021 鐵人賽 – Cloud)"
date: 2021-10-08T19:07:00+08:00
draft: false
url: "/2021/10/08/day23-infrastructure-as-code/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

昨天介紹的 Deployment Manager 可以透過 GUI 與 Command Line 的方式管理資源，算是 GCP 中自己專有的功能。而 Google 也支援了 IaC 的常見軟體，例如 Terraform、Chef、Puppet、Ansible、Packer 等。

IaC (Infrastructure as code)，基礎架構即代碼，指的是透過程式碼來對各種基礎設備進行管理與配置，而上述的這些軟體也不僅適用於 GCP 中，可以基本上將相同的代碼做些微的修改，套用到其它的雲端服務商，例如 AWS 與 Azure 等，在不同服務商間轉換設備相對而言會非常的方便。

透過 IaC，可以減少繁複的操作一模一樣的事情，也可以批次化大量的管理設備，對於 DevOps 等環節而言會方便非常多。

以 Terraform 為例，通常我們會使用 YAML 檔案設定各種部屬的基礎設備資訊。但例如 VPC 與防火牆 Rule，這些 IP 位置則比較像是一個變數，這種時候我們可以透過 Jinja 的 Template Engine 進行設定。
