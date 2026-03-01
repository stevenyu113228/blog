---
title: Source (Try Hack Me Writeup)
date: 2021-08-16T13:21:00+08:00
draft: false
url: "/2021/08/16/source-try-hack-me-writeup/"
categories:
  - 滲透測試
  - Try Hack Me
showToc: true
TocOpen: false
---

> 
URL : https://tryhackme.com/room/source  
IP : 10.10.63.154

## Scan

- 掃 Port`rustscan -a 10.10.63.154 -r 1-65535`![](/uploads/2022/02/7dd97-I6kP0We.png)
- 22
- 10000`nmap -A -p22,10000 10.10.63.154`- ![](/uploads/2022/02/85a9c-XF4bSvw.png)
- 發現 10000 是 `MiniServ 1.890`直接連上去- http://10.10.63.154:10000/
- ![](/uploads/2022/02/ea2f1-cd1cvG2.png)發現它有綁 SSL 要用指定網址才能進`10.10.63.154 ip-10-10-63-154.eu-west-1.compute.internal`
- 綁上 `/etc/hosts`再次訪問- ![](/uploads/2022/02/f0aea-yyal9X9.png)

## Exploit

- 透過版本可以查詢到 `MiniServ 1.890` Exploithttps://github.com/foxsin34/WebMin-1.890-Exploit-unauthorized-RCE/blob/master/webmin-1.890_exploit.py
- `wget https://raw.githubusercontent.com/foxsin34/WebMin-1.890-Exploit-unauthorized-RCE/master/webmin-1.890_exploit.py`
- `python3 webmin-1.890_exploit.py`![](/uploads/2022/02/bda70-MJV55av.png)
- ㄟ就直接拿到 root 了ㄟ
- 太無聊ㄌㄅ = =懶得戳 Reverse shell ㄌ- 直接用這個類似 Webshell 的東西 早點打完收工ㄅ
- ![](/uploads/2022/02/a3051-DBSTDZg.png)
- Root Flag`THM{UPDATE_YOUR_INSTALL}`取 User Flag- 先看 home 使用者名稱![](/uploads/2022/02/a55c7-Cq4YNie.png)看使用者資料夾- ![](/uploads/2022/02/54905-RRdc2bA.png)取得 Flag- ![](/uploads/2022/02/62d0d-E7Dcn37.png)
