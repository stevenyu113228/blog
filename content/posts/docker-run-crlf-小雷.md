---
title: Docker Run CRLF 小雷
date: 2022-10-13T20:49:35+08:00
draft: false
url: "/2022/10/13/docker-run-crlf-小雷/"
categories:
  - 廢文
showToc: true
TocOpen: false
---

在使用 Docker 時，偶爾會順手帶上 `-it` (interactive + Allocate a pseudo-TTY)，但我卻因此踩到了一個小雷。

![](/uploads/2022/10/image.png)

在 alpine 底下，如果加上了 `-it` ，則 `\x0a` 會被 Replace 成 `\x0d\x0a`。所以如果程式的輸出結果是 binary 的話，會因此壞掉 QQ
