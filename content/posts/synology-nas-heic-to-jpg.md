---
title: Synology NAS HEIC to JPG
date: 2022-09-17T13:20:44+08:00
draft: false
url: "/2022/09/17/synology-nas-heic-to-jpg/"
categories:
  - 廢文
showToc: true
TocOpen: false
---

iPhone 透過 Synology 的 Photo Station 備份，預設會存 HEIC，除非到 iPhone 相機的設定部分修改成存 JPG。 但 HEIC 在 Windows 平台上非常不友善 QQ。

目前貌似沒有看到 S 牌有提供什麼自動轉檔的程式，所以短期可能只能自己處理。

在 Linux 平台上，可以使用 [GitHub - shellbro/dockerfile-heif-convert](https://github.com/shellbro/dockerfile-heif-convert) 這套 Docker Image 來處理。

我們觀察這個 Docker 執行的 Script 可以發現，[convert-all](https://github.com/shellbro/dockerfile-heif-convert/blob/master/convert-all) 其實就是一個 for 迴圈去枚舉 *.heic，不過 iPhone 預設的副檔名是大寫的 *.HEIC，所以我們有兩種解法。 1. 修改現有的附檔名， 2. 修改這個 Script。 因為懶得重 Build Docker image 我選擇了 1 的方法。

腳本如下，先 ssh 上 NAS，並 cd 到準備轉檔的資料夾即可

```bash
mkdir heic
for a in *.HEIC; do mv -- "$a" "$(pwd)/heic/${a%.HEIC}.heic"; done
sudo docker run --rm -u "$(id -u):$(id -g)" -v $(pwd)/heic:/input -v $(pwd):/output shellbro/heif-convert
```

如果遇到權限問題，可以自己 hard code uid 跟 gid 即可，可以直接 `cat /etc/passwd | ` 取得 ID
