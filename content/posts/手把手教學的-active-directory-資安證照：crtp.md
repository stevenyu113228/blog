---
title: 手把手教學的 Active Directory 資安證照：CRTP
date: 2023-09-25T17:43:33+08:00
draft: false
url: "/2023/09/25/手把手教學的-active-directory-資安證照：crtp/"
categories:
  - 資訊安全
  - 滲透測試
showToc: true
TocOpen: false
---

我在今年七月時，報名了 CRTP 的證照，CRTP 的課程，全名 Attacking & Defending Active Directory Lab。 它的特色主要在於，所有題目都是 Windows 最新版本 Windows Server 2022 且 Windows Defender 全開的機器，Lab 以及考試的內容都圍繞在 Windows AD 之中。

它的 Lab 宗旨主要是對於 Windows 中，無法透過 Patch 修復的一些 Misconfiguration，相較於 OSCP 常見的打 CVE，方向不太一樣。另外就是，這張證照完全不需要使用到任何的 Linux 機器，例如 Kali。它的起步點預設於我們已經拿到一台 Domain Joined 的 Windows，並以那台 Windows 電腦作為起跑點，進行橫向移動，拿到 Domain Admin。

其實這張證照我在今年初就有打算想要報，不過後來看到了一些相關的通知，說 Pentester Academy 在 2023 年初將不再提供 CRTP 的證照。好在後來這張證照改由 Altered Security 繼續推出，看起來像是這堂課的講師 Nikhil 自己出來自立門戶，開了一間公司。

在今年七月初，當完了研發替代役的兵之後，距離公司的下一個大 Project 開始之間，有差不多一個禮拜的空擋，因此我就花了一週的時間把 Lab 全部刷完了 XD。

## 報名

CRTP 現在的報名頁面是以下網址：https://www.alteredsecurity.com/adlab

我覺得網站非常簡陋，看起來也很像釣魚網站，目前它有三種不同的方案，分別是 30 天 249 鎂；60天 379 鎂以及 90 天 499 鎂。

雖然我覺得 30 天就很夠用了，不過聽說裡面的 Lab 很適合拿來做一些自己的實驗，因此最終我選擇報名了 499 的 90 天 Lab。

報名結束後，系統就會自動寄 Email 並提供系統的平台網址，而平台網址內即可下載到課程的講義與影片檔。這邊與 OSCP 比較不一樣的點在於，這 90 天的 Lab 並不會立刻啟用，而是在未來 90 天內隨時都可以啟用。 這樣的優點在於，學生可以先看影片及教材上課，等到 Ready 後再 Email 通知官方開啟 Lab，開啟後才開始計算 90 天的 Lab，相對之下會比較沒有壓力。

但我……思考了一下，既然都買 90 天的 Lab 了，應該是綽綽有餘，因此報名後就立刻開啟了 Lab。

## 教材

CRTP 官方的教材可以分成三種形式，第一種是課程的影片，看起來是 CRTP 之前有辦過直播課程的錄影檔，4 個錄影檔案中，每一個約 4 小時。不過因為講師是印度人，口音有一點重，我覺得聽起來有一點點吃力。寂寞波大大在 Blog 分享過，他之前使用的方法是，把影片上傳到 Private 的 Youtube 中，並且等待 Youtube 自動產字幕。但我覺得，把官方的影片丟到網路上還是不太好，因此我的方法則是直接把影片丟給自己 Local 的 Whisper 來轉字幕，不過後來我還是懶得看了 XDD。

另外還有提供一份約 300 頁的投影片，投影片上的內容就是前面提到的影片內容，我覺得投影片的章節沒有切得非常好，但每一個章節都會對應到 Learning Objective。整份教材中總共有 22 個 Learning Objective，而每一個 Learning Objective 大約有 2 個左右的小題目，最終總共有 40 個 Flag。

這邊的 Flag 與標準的 CTF 不太一樣，比較像是問答題的感覺，可能是問某個 hostname / username 或是某個帳號的 hash、可以 abuse 的 service 的名稱等。

另外一個跟 OSCP 比較不一樣的點是， CRTP 的所有 Learning Objective 的 Lab 都有提供手把手的 Lab Writeup，總共有約 100 頁的 PDF 可以進行參考，基本上只要無腦的照著做，所有題目都可以迎刃而解。

## Lab

Lab 中總共約有 13 台機器，屬於多個 AD 的 Domain 與 Forest，因此題目也有一些 Cross Domain 的 Attack，其實我拿到 Lab 後，第一件事情是，直接開打 XD。完全沒有管到教材上面教的內容，直接把整個 Lab 當作 OSCP 的 Lab 一樣對待。

花了差不多五天的時間，我就順利的把整個 Lab 跟 Learning Objective 給刷完了，但總有一種不踏實感，感覺怪怪的。雖然打完了，但好像沒有學到太多東西？

接著我才回去看了課程的 Slide 以及 Lab 的教學，發現，我打從一開始就打歪了，快速地拿到了 Golden Ticket 飛後回去撿各個 Flag。雖然這種方法可以解題沒錯，不過我卻因此喪失了滿多學東西的機會。

如果依照 Lab 手把手的 Writeup，會發現每一個橫向移動、Exploit 方法，教材中都會教不同的手法，例如使用 PS-Remoting、檔案不落地執行 Powershell ……等等。而我的手法則是很暴力的 evil-winrm 進去關 AV、跑 mimikatz 進行 dump。

## 知識點總結

我有大致的整理了在 Lab 中的知識點，詳情可以到我的 Github (https://github.com/stevenyu113228/My-Security-Resources/blob/master/AD_Cheat_Sheet.md) 上查看，以下列舉一些大標題供大家參考。

- Evasion

PowerShell Execution Policy Bypass
- AMSI Bypass
- AV Bypass
- Loader
- Bypass CLM, APP Locker

Remote Control
- Powershell Remote
- winrs

Domain Enumeration
- User
- Group
- Trusted Domain
- ACL

Mimikatz
- Ekeys
- Lsa
- Over pass the hash
- Golden ticket
- Silver ticket
- DC Sync
- SID History
- Trusted Key
- Skeleton Key

Rubeus
- Over pass the hash
- Silver Ticket
- Diamond Ticket
- Kerberosting

Persistent
- Abuse DSRM
- SDDL
- RACE

Delegation
- Unconstrained Delegation
- Constrained Delegation
- Resource Based Constrained Delegation

AD CS
- ESC1
- ESC3
- ESC6

MSSQL
- Power SQL
- SQL Server Link Crawl

Blood Hound

## 考試

雖然我花了 5 天的時間刷完 Lab，再花了約一週的時間把所有教材的內容都練習完畢，正當我準備考試時，迎面而來的是公司這邊出現了超忙的 Project。因此拖到了九月中旬，我才選了一個週末進行考試。

CRTP 的考試如同 OSCP 一樣，是 24 小時的實戰考試，比較不一樣的點在於，它不需要監考，也不需要提前預約，只需要想考試時直接到平台上按下即可。按下按鈕後，系統約需要 10 分鐘的時間建立考試環境，接著就開考了。

簡單來說，考試總共有 6 台機器，包含第一台提供的學生機，從本地提權開始，進行各種 Domain Enumeration。到各種的 Delegation，最後拿到 Domain Admin 並打 Cross Domain 相關的攻擊，成功的 Compromise 5 台電腦。 整體上我花了 2 個小時打完，並且再花了 3 個小時寫報告。

報告沒有規定的格式與 Template，也不像 OSCP 等考試中會有提供 Flag，這邊就只需要成功地進入電腦後，下 hostname 以及 ipconfig 進行截圖即可。題目也沒有強制一定要提權，也有一些題目預設就是不能提權的，因此只需要進入電腦即可。

另外值得一提的是，官方有說，可以在報告上備注非英語母語者，這樣的話比較不會去叼英文的文法。我個人是邊打邊寫筆記，接著直接把筆記丟給 ChatGPT 要它幫忙生報告，這也是一種不錯的方法！

而報告是自己產成 PDF 後，直接透過 Email 寄給官方。因為我打題目的過程中，就使用 Markdown 來做筆記，因此我使用 OSCP 的 Markdown Template ([https://github.com/ret2src/OSCP-Exam-Report-Template-Markdown#docker](https://github.com/ret2src/OSCP-Exam-Report-Template-Markdown#docker)) 來寫完整的報告。

最終我在週六(9/16)晚上送出報告，並於週四(9/21)中午順利拿到了證照！

![](/uploads/2023/09/upload_3790957206e2e9cfbf09a8be93dd8751-1024x791.png)

## 心得

我覺得比起 OSCP， CRTP 的考試題目非常的友善，完全沒有任何的兔子洞，基本上看到哪裡有問題，就是直接往那邊打，所以這張證照的難度不是很高。另外關於鑑別度的部分，因為考試不需要任何監考，有非常大的作弊可能性。不過目前台灣幾乎沒有公司認這張證照，也沒有出現在資安署認可的清單中。因此我覺得應該也不會有人特別因此作弊吧？

嚴格來說，我覺得 CRTP 的價值在於它的 Lab 以及教材，證照部分可以把它理解成修課證明的感覺。其實我覺得任何證照的價值都不該是它本身的含金量，而是透過學習了這張證照的教材後，能為自己的工作或專業上提供怎麼樣的幫助，這才是最重要的！

另外一方面，CRTP 官方的客服也非常讚，對於題目有任何不清楚的地方，或是操作上遇到障礙，都可以直接用 Email 詢問客服，我其中詢問了四五次，包含了遇到題目故障，或是使用一樣的 Payload 卻打不出來…..等，客服們都在 30 分鐘內確認問題，並給出明確的解法，這一點在 CPENT 上是完全沒有發生的，客服別說 30 分鐘了，連 30 天都不回應我 QQ

另外，可以明顯的感覺出來 Altered Security 很努力想要走社群相關的推廣，只要通過考試，Email 會詢問考生的 Twitter 以及 Linkedin，官方也會直接 Tag 使用者發文恭喜。  
![](https://hackmd.io/_uploads/SJB8QCRk6.png)

![](/uploads/2023/09/upload_b1f4cadf331ea77e3d9fb905d961c35c-800x1024.png)

總結來說，對於 PowerShell 0 基礎，或是 AD 0 基礎的人而言，我非常推薦大家來考 CRTP 的證照，透過 Lab 的學習，可以學到很多東西！比起 OffenSice Security 的證照，無論是價格跟難度都友善很多，學習起來也比較沒有壓力，我覺得未來有空的話，我應該還會嘗試找 Altered Security 的其他證照課程繼續學習！
