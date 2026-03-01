---
title: "[Day3] Practice Resources (2021 鐵人賽 – PT)"
date: 2021-09-18T18:32:00+08:00
draft: false
url: "/2021/09/18/day3-practice-resources/"
categories:
  - iTHome 鐵人賽
  - 我想學滲透測試喵喵喵喵！！！！
showToc: true
TocOpen: false
---

CTF 通常會依照各種領域區分， Web 、 Reverse 、 Pwn 、 Crypto 等。而滲透測試如果一定要區分在這些領域的話，應該會比較偏向 Misc。滲透測試通常都會是以組合技為主，所以 CTF 的各種領域最好都要有基礎的了解。

## CTF 資源

以新手導向的 CTF 資源，我推薦 [picoCTF](https://picoctf.org/)，上面的題目都非常的新手友善，關於 picoCTF 的解題內容，也歡迎觀看隔壁棚，我的隊友 Yvonne 的[鐵人賽文章](https://ithelp.ithome.com.tw/users/20134305/ironman/4222)。

除了 picoCTF 之外，對於初學者學習 CTF 類資源，又與 PT 叫相近的，也推薦大家可以使用 [RootMe](https://www.root-me.org)。

## 靶機資源

有了基礎的 CTF 各領域技術後，接下來就可以進入練習滲透測試技能的階段。在這邊，我主要推薦四大平台：

- [TryHackMe](https://tryhackme.com/)TryHackMe 是這三個網站中，筆者最推薦的一個平台，上面有數以百計的 Linux 與 Windows 機器，也有一系列的滲透測試相關教學，可以依照著指示與提示，手把手的帶著大家解題，算是三者中對於新手最友善的平台。
- 其免費版就有不少的機器可以使用，而付費版則有更多課程與靶機。感謝 飛飛 ㄉ錢包贊助 QAQ。[Vulnhub](https://www.vulnhub.com/)- Vulnhub 上有許多靶機的虛擬機 ova 檔案，可以部屬在自己的電腦中
- 不過對於電腦效能有限的人可能會有一點不友善
- 另外也可以將 ova 檔案部屬到雲端平台上進行練習[HackTheBox](https://www.hackthebox.eu/)- HackTheBox 是網路上最多人推薦的平台，但筆者覺得他也有些許的缺點 QQ
- 其上面免費版的靶機通常都會是多人共用，(數人到數十人不等)很多時候會造成互相的干擾，除非使用 `VIP+` 的方案他是一個綜合性的資安網站，除了滲透測試的靶機之外，也有許多軟硬體相關的 CTF 題目

在此我建議，各平台的靶機，如果有機會都可以去嘗試看看。如果只選擇一個平台的，一定會遇到一些的盲點。本次鐵人賽的內容前半主要會以 TryHackMe 上的靶機為主，預計後半可能會刻金買 HackTheBox 來玩。

## 證照資源

與滲透測試相關的證照，最有名的就是 [OSCP](https://www.offensive-security.com/pwk-oscp/) 證照，是由 Offensive Security 提供的 Penetration Testing with Kali Linux (PEN-200)。 它算是少數資安領域中的實做型證照，考試內容為在 24 小時內打下 5 台指定的靶機，並取得 root / System 權限，非常推薦大家對於滲透測試有了一定的認識後報考。

OSCP 官方有提供 PWK 的 Labs，其 30 天的 Lab 使用權搭配一次的考試，價格為 999 美元；60 天 Lab 使用搭配一次考試 則是 1199 元；90天的 Lab 搭配一次考試為 1349 元。
