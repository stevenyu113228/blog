---
title: Lame (Hack The Box Writeup)
date: 2021-09-05T13:59:00+08:00
draft: false
url: "/2021/09/05/lame-hack-the-box-writeup/"
categories:
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

> 
URL : https://app.hackthebox.eu/machines/Lame

IP : 10.129.197.50

## Info gathering

- Port Scanning![](/uploads/2022/02/3a51c-aqhaBG8.png)21,22,139,445,3632![](/uploads/2022/02/188e9-4nP95qL.png)- Anonymous FTP
- SMB
- 3632 Port distccd

## File Protocol

- Try anonymous login FTP![](/uploads/2022/02/59c35-4SbeeBH.png)
- It's emptyTry anonymous login SMB- ![](/uploads/2022/02/6da0b-tAFlRJ1.png)
- There are `tmp` and `opt` folder
- Access tmp folder![](/uploads/2022/02/633dc-tM6RvLY.png)
- ![](/uploads/2022/02/cc75d-nfdeBLb.png)Download all file

## Exploit Distccd

- Distccd_rce_CVE-2004-2687https://gist.github.com/DarkCoderSc/4dbf6229a93e75c3bdf6b467e67a9855
- ![](/uploads/2022/02/93f57-Q1Y4t7j.png)Run Reverse shell- ![](/uploads/2022/02/3fef3-mtPM4ei.png)
- ![](/uploads/2022/02/a8f49-83ysiHm.png)Get User Flag- ![](/uploads/2022/02/ed168-40sakTp.png)

## Privilege escalation

- Run LinPEAS![](/uploads/2022/02/8ecab-yxrx9q6.png)NFS Exploit?![](/uploads/2022/02/2b339-tp6ptzW.png)- Eterm SGID Binary?![](/uploads/2022/02/8826c-zAYCC6R.png)- nmap SUID !!Nmap GTFOBins- Shell (2) Interactive shellhttps://gtfobins.github.io/gtfobins/nmap/#suid
- `nmap --interactive`
- `nmap> !sh`![](/uploads/2022/02/d558f-Gw41QSr.png)Get Root Flag- ![](/uploads/2022/02/bfccc-WXIvv23.png)
