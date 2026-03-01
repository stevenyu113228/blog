---
title: 使用 Docker 執行 Metasploit
date: 2022-03-04T22:31:31+08:00
draft: false
url: "/2022/03/04/使用-docker-執行-metasploit/"
categories:
  - 資訊安全
  - 滲透測試
showToc: true
TocOpen: false
---

其實，我跟 Metasploit 沒有太熟，因為他太方便，太無惱了，導致 OSCP 的考試禁止使用。 我在練習 PT 時也就都直接當作這個軟體不存在 XD。 但事實上 Metasploit 真的算是一個很實用的工具，只需要無腦的設定好 payload 之後 run / exploit 下去就好了。

而 Reverse Shell 的部分，我一直以來都習慣使用 Stageless 的 Shell，不過其實 Meterpreter 以及其他 Staged 的 Shell 在實務上是滿好用的。

現在常見的滲透測試 OS ，例如 Kali 或是 Parrot 裡面都已經內鍵了 Metasploit 的指令，使用者不需要煩惱太多安裝的問題。通常在各種平台練習靶機時，官方都會發放一個 VPN， 攻擊者只需要把 Shell 彈回這個內網裡面的自己機器即可。

不過在實務上，事情就沒有這麼簡單了，如果我們使用簡單的 Stageless 的 Reverse shell ， 當不在內網時，我們可以在雲端機器上隨便開一個 nc / ncat 來收，但 Stage 的就不同了，通常會需要使用 `msfconsole` 的 `exploit/multi/handler` 來接收。

在 GCP 上沒有內建 Kali, Parrot OS 的機器 ([AWS 上有 Kali](https://aws.amazon.com/marketplace/pp/B08LL91KKB)，但我比較不習慣用 AWS)，當然可以自己上傳 image 檔案上去做一些處理，不過先前我的經驗是會遇到一些雷需要克服，因此最好的方法還是直接在雲端機器裡面部屬  msf，根據官方的說法，可以透過下面指令來安裝 msf

```bash
curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb > msfinstall && chmod 755 msfinstall && ./msfinstall
```

但總覺得這種方法有一點噁心，電腦裡面被裝一些有的沒的東西，還有更大的一個問題是，我試著照做之後，電腦就卡了 20 分鐘， 卡在 `ruby msfdb init` 的指令上面。

## Docker Metasploit

不想讓電腦的環境變髒，最容易會想到的就是 Docker 了，找了一下，官方也真的有推出 msf 的 Docker，可以透過一行指令就乾淨的安裝完畢！

```bash
sudo docker pull metasploitframework/metasploit-framework
```

而執行時，也只需要輸入這一行，其中 Port 的部分需要手動設定需要開的 Port，例如我希望用 5487 Port 來收 Shell 就可以這樣打

```bash
sudo docker run --rm -it -p 5487:5487 metasploitframework/metasploit-framework
```

接下來就跟常規的使用方法一樣，就不特別贅述了。

![](/uploads/2022/03/2022-03-04-22-29-23.png)

值得一提的是如果使用雲端平台，雲端的 Port 也需要記得開。
