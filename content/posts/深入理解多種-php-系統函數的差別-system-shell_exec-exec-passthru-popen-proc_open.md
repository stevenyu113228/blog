---
title: "深入理解多種 PHP 系統函數的差別 (system, shell_exec, exec, passthru, popen, proc_open)"
date: 2022-03-19T13:57:50+08:00
draft: false
url: "/2022/03/19/深入理解多種-php-系統函數的差別-system-shell_exec-exec-passthru-popen-proc_open/"
categories:
  - 廢文
  - 網頁安全
showToc: true
TocOpen: false
---

在戳 Webshell 時，時常會被 Disable Function 給雷，而網路上的各種 Cheat Sheet 也常常會教我們， Bypass Disable Function 的其他函數。這邊先不考慮 LD_PRELOAD 或是其他奇技淫巧的繞過方法，我們從最常見的 6 種可執行系統指令的 Function 開始，探討它們在正常使用時的不同。明明功能都差不多，甚至一樣，為什麼 PHP 要定義出這麼多的函數呢？ ~~因為 PHP 是一個非常 Hacker Friendly 的語言，有各種方法可以讓駭客繞繞繞！~~~

本文會比較 `system`、`shell_exec`、`exec`、`passthru`、`popen` 以及 `proc_open` 等 Function 的差異。

## system

讓我們從 `system` 函數開始，觀察 [Spec](https://www.php.net/manual/en/function.system.php) 可以看出，官方的敘述。

> 
system — Execute an external program and display the output

system 指令會執行外部程式，並且直接把結果輸出 (類似於 `echo` 到螢幕上)，這邊也有一個特性就是，當程式每輸出一行，畫面結果就會刷新一次 (儘管程式可能還沒結束)。

`system` 指令有兩個參數，分別是 `$command` 以及 `&$result_code`。`$command` 應該不用特別敘述，而 `&$result_code` 會使用 Pass by reference 方式把 Linux 的 Return Status Code 給回傳到該參數。

而 system 函數的回傳值，如果指令執行成功，會回傳 **最後一行** 的值，如果程式執行不成功，會回傳 fasle。

假設我們寫一個

```php
   INIT_FCALL                                               'system'
          1        SEND_VAL                                                 'whoami'
          2        SEND_REF                                                 !1
          3        DO_ICALL                                         $2      
          4        ASSIGN                                                   !0, $2
          5      > RETURN                                                   1

branch: #  0; line:     2-    2; sop:     0; eop:     5; out0:  -2
path #1: 0,
```

可以得知 system 是透過 `INIT_FCALL` 串 `DO_ICALL` 進行執行的，也就是他是 PHP 底層透過 OP-Code 執行的 Function。

## shell_exec

一樣從官方 [Spec](https://www.php.net/manual/en/function.shell-exec.php) 開始

> 
shell_exec — Execute command via shell and return the complete output as a string.

`shell_exec` 只有一個參數為指令，也只有一個回傳值，是完整的輸出的 String，而且他不會在執行時回傳執行結果於螢幕上。

範例程式碼

```php
   INIT_FCALL                                               'shell_exec'
          1        SEND_VAL                                                 'cat+%2Fetc%2Fpasswd'
          2        DO_ICALL                                                 
    3     3      > RETURN                                                   1
```

shell_exec 與 system 一樣，都是先透過 `INIT_FCALL` 初始化，再使用 `DO_ICALL` 進行執行的動作，也都屬於 PHP 最底層的 Operation。

值得一提的是，PHP 的

```
``
```

這種符號其實只是 `shell_exec` 的語法糖，可以用 vld 進行觀察

```php
   INIT_FCALL                                               'shell_exec'
          1        SEND_VAL                                                 'whoami'
          2        DO_ICALL                                         $0      
          3        ECHO                                                     $0
          4      > RETURN                                                   1

branch: #  0; line:     2-    2; sop:     0; eop:     4; out0:  -2
path #1: 0,
```

## exec

下一個來介紹 exec， 一樣從 [spec](https://www.php.net/manual/en/function.exec.php) 開始。

> 
exec — Execute an external program

exec 的參數就比較多了，分別有 `$command`, `&$output` 以及 `&$result_code`。預設 `exec` 的 return 值跟 system 一樣，會是最後一行。如果需要捕獲完整的 output ，則需要使用 `$&output` 讀取 reference 的方法，而它預設會回傳一個 array，它是輸出透過 `\n` 分隔的結果；預設 `exec` 的結果也不會顯示於螢幕上。

範例 Code

```php

  string(37) "PING 8.8.8.8 (8.8.8.8): 56 data bytes"
  [1]=>
  string(55) "64 bytes from 8.8.8.8: icmp_seq=0 ttl=111 time=2.624 ms"
  [2]=>
  string(55) "64 bytes from 8.8.8.8: icmp_seq=1 ttl=111 time=2.663 ms"
  [3]=>
  string(31) "--- 8.8.8.8 ping statistics ---"
  [4]=>
  string(57) "2 packets transmitted, 2 packets received, 0% packet loss"
  [5]=>
  string(58) "round-trip min/avg/max/stddev = 2.624/2.643/2.663/0.000 ms"
}

0
```

vld 觀察，也是一樣的結果

```
Finding entry points
Branch analysis from position: 0
1 jumps found. (Code = 62) Position 1 = -2
filename:       /mount_point/demo_code/exec_test.php
function name:  (null)
number of ops:  7
compiled vars:  !0 = $ret, !1 = $output, !2 = $result_code
line      #* E I O op                           fetch          ext  return  operands
-------------------------------------------------------------------------------------
    2     0  E >   INIT_FCALL                                               'exec'
          1        SEND_VAL                                                 'ping+-c+2+8.8.8.8'
          2        SEND_REF                                                 !1
          3        SEND_REF                                                 !2
          4        DO_ICALL                                         $3      
          5        ASSIGN                                                   !0, $3
    9     6      > RETURN                                                   1

branch: #  0; line:     2-    9; sop:     0; eop:     6; out0:  -2
path #1: 0,
```

## passthru

從 [Spec](https://www.php.net/manual/en/function.passthru.php) 開始。

> 
passthru — Execute an external program and display raw output

它會直接把值(包含 binary 資料)給吐出來，預設沒有回傳值，傳入參數為 `$command` 與 `&$result_code`。

範例扣

```php
   INIT_FCALL                                               'passthru'
          1        SEND_VAL                                                 'cat+%2Fbin%2Fbash'
          2        DO_ICALL                                                 
          3      > RETURN                                                   1

branch: #  0; line:     2-    2; sop:     0; eop:     3; out0:  -2
path #1: 0,
```

## popen

[Spec](https://www.php.net/manual/en/function.popen.php).

> 
popen — Opens process file pointer

透過 popen 會開啟一個 handler，可以設定 `w` 與 `r` 權限，並可以透過 `fread` 等方式進行讀取，讀取完畢後需要使用 `pclose` 進行關閉。

```php
   INIT_FCALL                                               'popen'
          1        SEND_VAL                                                 '%2Fbin%2Fls+-al'
          2        SEND_VAL                                                 'r'
          3        DO_ICALL                                         $2      
          4        ASSIGN                                                   !0, $2
    3     5        INIT_FCALL                                               'fread'
          6        SEND_VAR                                                 !0
          7        SEND_VAL                                                 2096
          8        DO_ICALL                                         $4      
          9        ASSIGN                                                   !1, $4
    4    10        INIT_FCALL                                               'pclose'
         11        SEND_VAR                                                 !0
         12        DO_ICALL                                                 
    6    13        ECHO                                                     !1
```

## proc_open

[Spec](https://www.php.net/manual/en/function.proc-open.php)

> 
proc_open — Execute a command and open file pointers for input/output

proc_open 類似於 popen，不過它的功能又多更多了

```
proc_open(
    array|string $command,
    array $descriptor_spec,
    array &$pipes,
    ?string $cwd = null,
    ?array $env_vars = null,
    ?array $options = null
): resource|false
```

`$command` 可以是 array 或 string 的格式；而 `$descriptor_spec` 可以指定 STDIN, pipe 或 socket。

`$cwd` 可以指定執行時的絕對路徑，如果不給予則會是預設路徑；`$env_var` 則可以給予環境變數，而 `$options` 則一些奇怪的功能，目前都是 Windows Only。

好麻煩喔，我有點懶得做實驗了，這個等需要用到時再說吧！

## 結論

日常最常用到的 `system` ，會直接把輸出吐到螢幕上，但它的回傳 string 只會是最後一行；如果需要獲得完整的回傳 string，並且不把結果回傳到螢幕的話，可以用 `shell_exec` 來接； `exec` 可以用 reference 的方法來接完整的輸出 Array 格式，預設也不會顯示到螢幕上；`passthru` 會顯示到螢幕，且支援 binary 的值，但無法把回傳的東西放入變數中。而 `popen` 以及 `proc_open` 則有一些進階用法可以把輸入、輸出結果串上 `pipe` 或 `descriptor` 中，而最最進階版的是 `proc_open`，可以指定最多的參數。
