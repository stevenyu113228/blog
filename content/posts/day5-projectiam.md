---
title: "[Day5] Project,IAM (2021 鐵人賽 – Cloud)"
date: 2021-09-20T18:53:00+08:00
draft: false
url: "/2021/09/20/day5-projectiam/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

今天要來介紹的是，剛進入雲端一定會面對到的部分：Project以及 IAM。我覺得這個相關的主題會是雲端裡面數一數二無聊的 QQ，但以一個完整的雲端架構而言，這卻是非常非常重要的部分。特別是 IAM 這個主題，單純這個主題，事實上身為一個雲端管理者、架構師，就可以上一整個月的課程了。

## Project

任何的 GCP 服務，都會屬於一個 Google Cloud Console 的 Project，無論是管理 API 、 管理計費方式、增加其他的協作者等都是基於 Project 為單位，而一個 Project 底下則可以有許多的資源 (Resources)。

一個 Project 可以包含非常多的使用者，且每個 Project 都是一個獨立的單位，獨立的隔間，他們會分開的計費。

每一個 Project 會有三個相關的識別屬性

- Project ID可以自己定義，它必須是 Global Unique 的
- 設定完後就不可更改Project Name- 可以自己定義，不需要 Unique
- 設定完後仍然可以再次修改Project Number- 不可自行定義，由 GCP 自動產出
- 不可自行更改

### Project 階層式管理

GCP 也可以透過階層式的進行管理，以一個公司為例，通常會使用組織節點 (Organization Node)，而一個公司底下可能會有不同的部門，這個時候就可以透過資料夾 (Folders) 進行管理，而每一個部門底下也可能會有多個 Project；每個 Project 底下則又可以有多個 Resources，範例如下圖所示。

![](/uploads/2022/02/bicqdtS.png)

> 
圖片來源：[Google Cloud Creating and managing Folders](https://cloud.google.com/resource-manager/docs/creating-managing-folders)

不同的資料夾間，可以設定不同的 IAM 權限管理政策，通常來看，整個公司架構的樹狀圖中，位居越高位的管理權限越大，而越下面的權限越低。

## IAM

Identity and Access Management (IAM)，身分以及訪問權限管理，是雲端管理之中，最最重要的一件事情。如果一個 IAM 的階層管理做得越完善，越齊全，就越不可能會遇到一些看起來[很荒謬的事情](https://today.line.me/hk/v2/article/PvKqVV)。

例如上述的新聞中，抖音公司的實習生誤刪了資料庫，包含了 ML 平台的 Batch 模型全部都被誤刪。先不考慮官方要嫁禍給實習生的部分，如果說新聞真正屬實的話，那我們就可以斷定說抖音公司的各種 Policy 可能設計的不夠完善。

在訂定各種權限控管時，我們最需要注意的重點是，權限能設定的越小越好，負責 A 專案的人就只能修改，更動 A 專案，盡量以不讓他接觸到其他不屬於他的資源為原則。

GCP 在 IAM 的定義上依照了 3 個元素。`Who` `can do what` `on which resource`。

### Who (誰)

在 GCP 的 IAM Policy 當中，有 4 種 `Who` 的定義，

- Google Account or Cloud Identity userGoogle Account 是大家常見，習慣的 Gmail 帳號例如 `meow@gmail.com`Cloud Identity 是屬於公司、機構內的認證方式- 例如 `meow@meowmeow.com`Google Group 帳號- 例如 `meow@googlegroups.com`Cloud Identity or G Suite domain- 假設公司先前的架構有使用到微軟的 AD，或是 LDAP 等使用者管理，則可以用 Google Cloud Directory Sync 的技術，將使用者匯入為 Cloud Identity
- G Suite 相信大家都不陌生，公司、學校等機構都可以透過 G Suite 建立帳號與群組。Service Account- 我們也可以設定一個帳號是專門給 Services 使用的，例如說我想要設定一台 VM，當他自己覺得他不夠用時，就透過指令再開一台的 VM，在這種情形下，我們就需要設定一個 Services Account 給這台 VM ， 並給予他新增機器的權限。

### Can do what (可以做什麼事)

Can do what，可以做什麼事情，這邊我們又可以把一個 Role 分成了三個部分，Service (服務)、Resource (資源) 與 Verb (動詞)。

例如說我們有一台 Compute Engine 的雲端 VM instances，我們可以設定有一個 IAM 中的使用者，只能幫他進行開機，而不能進行關機以及刪除，這種時候我們就可以設定他的 Services 是 `compute` ； 他的 Resource 是 `instances`；而 Verb 則是 `start`，我們可以給予該使用者一個 Role 為 `compute.instances.start`，就能完成這樣的功能。

### On which resource (在什麼資源上)

我們可以將上述的 role 套用在指定的 Project 或進一步的資源內，或是各種的服務上，舉例來說，我可以設定使用者只能操作指定的 Project 內的指定資源，只能幫指定的雲端 VM 進行開機等。

### IAM 預設的 Role

相信大家看到這邊，一定覺得如果我公司有上百個人的話，一行一行的幫所有人寫 role 是一件很辛苦的事情，所以 Google 也提供了 3 種 IAM role 可以供大家快速套用。這邊的 Role 主要就是定義 "Can do what" 的部分。

#### Primitive Role

在 Primitive Role 中，我們可以把使用者分成 4 個部分。

- Owner (擁有者)可以做的事情最多，包含了……
- 邀請、刪除新成員
- 刪除 Project
- **以及所有編輯者可以做的事情**Editor (編輯者)- 編輯者可以……
- 部屬 Application
- 修改 Code
- 修改 Services 的設定
- **以及所有 Viewer 可以做的事情**Viewer (觀看者)- 觀看者就僅有唯讀的權限，不能進行任何的更動Billing administrator (計費管理員)- 計費管理員主要負責處理各種計費相關的服務
- 他可以增加與移除管理員

#### Predefined Role

相較於 Primitive Role，Predefined 的規則會更加的細緻一點點，我們可以不用把所有有編輯權限的人都設定為 Editor。例如說，我們可以選擇一個預設的 `InstanceAdmin` 的 Role，而這個 Role 就內建了許多精細的 Role。他內部包含了：

- `compute.instances.delete`
- `compute.instances.get`
- `compute.instances.list`
- `compute.instances.setMachineType`
- `compute.instances.start`
- `compute.instances.stop`
- ……等

#### Custom Role

而通常足夠大的公司，需要將權限分的夠細的話，就需要透過 Custom Role 了，可以設定到最細的，哪一個人，擁有怎麼樣的權限。

## 總結

今天的內容真的是又臭又長阿 QQ ，身為一個初學者，我們只需要知道，Google Cloud 的所有資源上面都要有一個 Project，而 Project 上可以用各種資料夾進行階層式的管理；每一個雲端的使用者都可以設定自己的雲端 Policy，而這些權限都要越小越好，以免發生意外，存取到不屬於他的資源；真的發生意外時，也可以讓災害降到最小。

今天的內容就到這邊，明天預計來跟各位介紹一下雲端最簡單最常用的資源：VM，虛擬機。
