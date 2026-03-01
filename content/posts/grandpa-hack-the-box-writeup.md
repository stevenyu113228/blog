---
title: Grandpa (Hack The Box Writeup)
date: 2021-09-15T13:54:00+08:00
draft: false
url: "/2021/09/15/grandpa-hack-the-box-writeup/"
categories:
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

- URL : https://app.hackthebox.eu/machines/13
- IP : 10.129.2.85

## Recon

- Rustscan![](/uploads/2022/02/ee070-fIUBhTc.png)
- 開 80 portnmap- 發現是 IIS6 可以用 CVE-2017-7269 RCE
- https://github.com/g0rx/iis6-exploit-2017-CVE-2017-7269

## Exploit

- 執行 Exploit![](/uploads/2022/02/e094c-i3boH3K.png)開 nc 收 shell- ![](/uploads/2022/02/85665-yPKDwiV.png)跑 systeminfo- ![](/uploads/2022/02/75b27-tbw8skf.png)發現 Windows 2003 可以跑 Churrasco- https://github.com/Re4son/Churrasco用 `impacket-smbserver meow .` 開- ![](/uploads/2022/02/2200f-MU2TXX1.png)
- 執行 Churrasco提權完畢- ![](/uploads/2022/02/51ccf-UrZVB2K.png)取得 User Flag- `bdff5ec67c3cff017f2bedc146a5d869`取得 Root Flag- ![](/uploads/2022/02/371e7-gQePLVE.png)
- `9359e905a2c35f861f6a57cecf28bb7b`
