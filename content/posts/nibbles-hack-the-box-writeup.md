---
title: Nibbles (Hack The Box Writeup)
date: 2021-09-15T14:02:00+08:00
draft: false
url: "/2021/09/15/nibbles-hack-the-box-writeup/"
categories:
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

- URL : https://app.hackthebox.eu/machines/121
- IP : 10.129.214.27

## Recon

- Rustscan![](/uploads/2022/02/b725c-m4G08Jy.png)觀察首頁- ![](/uploads/2022/02/4e75a-0jgF561.png)
- ![](/uploads/2022/02/5d1fd-RYElMsf.png)
- Apache
- ![](/uploads/2022/02/414a1-IQdi1Cj.png)找到隱藏目錄 `/nibbleblog/`看起來是一個 CMS- ![](/uploads/2022/02/c5168-KutA1td.png)搜尋 Exploit- ![](/uploads/2022/02/b2906-2Mdxscn.png)
- 看起來 4.0.3 版本有洞
- 有帳密就可以傳東西
- https://github.com/TheRealHetfield/exploits/blob/master/nibbleBlog_fileUpload.py繼續掃目錄- http://10.129.214.27/nibbleblog/admin/
- ![](/uploads/2022/02/959a6-cNJwr3I.png)
- ![](/uploads/2022/02/87ecb-azNBSX5.png)登入頁面![](/uploads/2022/02/223fb-PiHxr0E.png)- install![](/uploads/2022/02/be6fd-YejpqvW.png)- updateGoogle 到預設密碼- `admin / nibbles`
- ![](/uploads/2022/02/648e5-lrj8MCV.png)
- 這邊有點通靈 QQ

## Exploit

- 用 MSF`msf multi/http/nibbleblog_file_upload``spawn` shell- `python3 -c 'import pty; pty.spawn("/bin/bash")'`取得 User flag- ![](/uploads/2022/02/88ffb-qHqBpzs.png)

## 提權

- 起手式![](/uploads/2022/02/2ad8f-Ryh3MeA.png)可以執行 `monitor.sh`直接用`/bin/bash`蓋掉- ![](/uploads/2022/02/ed26f-I6ka7fH.png)sudo 執行- ![](/uploads/2022/02/f2295-gTwWLBu.png)
- 取得 Root取得 Root Flag- ![](/uploads/2022/02/d780e-xYnSpZj.png)
- `4c0ebef2dfb32a8d33212e1a902f50f2`
