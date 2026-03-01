---
title: "[Day4] Tools And Environment (2021 鐵人賽 – PT)"
date: 2021-09-19T18:33:00+08:00
draft: false
url: "/2021/09/19/day4-tools-and-environment/"
categories:
  - iTHome 鐵人賽
  - 我想學滲透測試喵喵喵喵！！！！
showToc: true
TocOpen: false
---

今天來介紹一下滲透測試常用的環境與工具，正所謂工欲善其事，必先利其器。準備好自己熟悉、習慣的作業系統與程式環境，對於後續的各種事情都可以事半功倍。

## 虛擬機

通常，對於滲透測試相關的設備與環境，我們都會使用虛擬機(Virtual Machines)，目前常見的軟體有 [VirtualBox](https://www.virtualbox.org/) 、 [VMWare](https://www.vmware.com/) 、 [Parallel Desktop](https://www.parallels.com/) 與 [Qemu](https://www.qemu.org/) 等。大家可以挑選自己習慣的環境，除了使用習慣上，基本上以初學者來講，虛擬化環境沒有太大的差別，如果各位仍沒有任何偏好的化，我會建議大家從 VirtualBox 開始。如果需要使用 VMWare 的話，也請務必採用付費的 Workstation 或 Fusion 版本，才能方便使用完整版的功能，例如 Snapshot

## 作業系統

作業系統方面，通常我會建議至少同時需要準備一個 Linux 的作業系統，以及一個 Windows 的作業系統，這樣比較能快速的應付各種的需求與測試。作業系統的版本與偏好，則也是看大家的習慣而定。

以 Linux 作業系統為例，許多人都會推薦使用 Kali Linux 與 Parrot OS，但其實我個人反而比較喜歡乾淨的 Ubuntu 作業系統， Kali,Parrot Linux 最大的特色與優點是，他集成了非常多常見的紅藍隊工具，可以讓使用者快速的上手，免去了各種安裝的麻煩；相對之下，Kali Linux 的缺點則是，它太像一個軍火庫了，裡面的工具過多，也常常得使新手產生混亂。而對於某些需要指定特殊版本的程式，在 Kali 上也有可能會出現一些小問題，但本次鐵人賽主要使用的 Linux 環境，我預計仍會採用 Kali Linux，因為可以省去許多安裝環境的麻煩。

而 Windows 環境就比較自由一點了，通常建議可以使用 Windows 10 的 Pro 版本以上，或是 Windows Server 的各種發行板，主要 Windows 的作業系統是用來處理一些，面對到 Windows Server 的靶機，我們可以在自己的 Windows 作業系統上做各種的實驗與演練，待測試完成後，再將指令與程式攻擊至靶機，以本次鐵人賽而言，預計都會以 Linux 的作業系統為主，如果想要手把手跟著做的朋友們，不需要急著安裝也沒有關係！

## 程式

很多人都在詢問，想要當一駭客到底要學會什麼程式語言，要學會哪些工具。我覺得，如果要成為一位厲害的資安技術人員，不太可能單靠學會幾樣的工具與程式語言就能應付所有的環境。關於程式方面，這邊大概列一些最最基本的軟體，主要提供給非使用 Kali Linux 作業系統的使用者，以利後續的其他工具安裝。

- Command Line ToolsGit下載工具或 Exploit 的 RepoPython2- 許多的 Exploit 仍然會使用到 Python2，所以一定要安裝！Python3- 另外有一些 Exploit 會使用到 Python3，所以兩種版本都要進行安裝Net- Netcat基本的網路連線工具，可以連接 Reverse，也可以進行檔案傳輸ftp- 基礎的 FTP 檔案傳輸工具。smbclient- 基礎的 SMB 檔案傳輸工具wget / curl- 基礎的下載檔案工具Reverse And Forensics- Objdump可以顯示檔案的組合語言Wireshark- 可以檢視封包內容Compiler- GCC、G++編譯 C/C++ 語言Make- 編譯程式使用

而對於真正攻擊相關的程式，預計會在後續實作階段，使用到時再來跟各位介紹，偶爾會在網路上看到許多攻擊程式的總整理，但在沒有一定的基礎，沒有實際演練過的情況下，通常看著一大堆的指令，初學者很容易不知所措。就算學會了指令與程式，也不知道可以運用在什麼地方。
