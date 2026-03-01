---
title: "[Day8] Storage (2021 鐵人賽 – Cloud)"
date: 2021-09-23T18:56:00+08:00
draft: false
url: "/2021/09/23/day8-storage/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

介紹完了虛擬機器與網路後，接下來要跟各位介紹的是關於雲端的儲存方案。Google Cloud 有許多雲端儲存的服務可以供大家選擇，例如 Cloud Storage 、 Cloud SQL 、 Cloud Spanner 、 Cloud Datastore 與 Cloud Bigtable。今天會與各位介紹最直觀，最直覺的 Cloud Storage。

## 簡介

Cloud Storage 主要可以用來儲存各種 Binary 的檔案，例如圖片檔、壓縮檔等。根據[新聞報導](https://www.bnext.com.tw/article/63705/apples-spending-on-google-cloud-storage-on-track-to-soar-50-this-year)，包含了蘋果、抖音等各大公司，背後都有使用到 Google Cloud 的 Storage 服務，蘋果更是使用到了 8 Exabytes 之多。如果沒有意外的話，Google Drive 背後就是運行 Cloud Storage 的服務。

Cloud Storage 預設會將所有資料進行加密，並使用 HTTPS / TLS 進行傳輸。可以取得 HTTPS 的直接連結，作為圖床使用。

Cloud Storage 不能算是一種檔案系統，它是透過 Bucket 的方式儲存不可變(immutable) 的檔案；不過也有第三方的工具 Cloud Storage FUSE 可以將 Cloud Storage 掛載成虛擬的網路硬碟。

## Class

Cloud Storage 分成了 4 種不同的 Class ， 分別是 Multi-regional 、 Regional 、 Nearline 與 Coldline，他們的價格與使用情境有許多的不同。

### Multi-regional / Regional

Multi-regional 與 Regional 屬於高性能的儲存方案，適合儲存馬上就須要使用的資料，它的存取費用低，而儲存費用高。

Regional 的 Class 會把資料儲存在指定的 Region 中 ， 如 asia-east1 等，他比起 Multi-regional 會相對比較便宜，但缺少冗餘。

Multi-regional 會透過地理冗餘，我們可以選擇一個地理位置，例如美國、歐洲、亞洲等。而資料會存在至少兩個地方，他們的地理位置距離至少差距 160 公里。假設其中一個地方發生大爆炸或戰爭(? ，其他地方還是可以存取到資料。

### Nearline / Coldline

Nearline 與 Coldline 被稱為備份、歸檔用存儲，它的存取費用高，而儲存費用低。適合存放數個月才會存取一次的檔案。

Nearline 適合儲存平均每個月存取一次，或更低頻率的使用情境。例如，不斷地將文件存到 Cloud Storage ，並每個月才須要將資料拿出來分析一次。

Coldline 則是比 Nearline 儲存價格更低的方案，它的適用情境下，資料的取用頻率也更低，每年最多只存取一次的情境。它有最短的存取期限為 90 天 ，適用於災害恢復等狀況使用。

## 資料傳輸方式

把資料送上 Cloud Storage，有三種方法，Online transfer、Storage Transfer Service 與 Transfer Appliance，來因應不同的使用需求。

- Online transfer透過自己的電腦，在 Cloud Console 上進行拖拉
- 或是使用 Cloud SDK 中的`gsutil` 等工具進行存取
- 透過 RESTful API 進行存取Storage Transfer Service- 可以自行排程 Batch 的資料傳輸，從其他的雲端服務商傳輸資料，例如 AWS 的 S3。Transfer Appliance- 透過實體的運送機架式的伺服器來傳輸資料，並透過貨運公司進行實體的運送。

這邊讓我想到一個酷東西，是 AWS 的服務 [Snowmobile](https://aws.amazon.com/tw/snowmobile/)。根據 AWS 官網所述， Snowmobile 是一個長達 45 英尺的貨櫃，透過聯結車來載運，每一個貨櫃可以載運 100 PB 的資料，也是超級超級大資料傳輸的解決方案，真的……超ㄎㄧㄤㄉ！

![](/uploads/2022/02/xfMM386-1024x513.png)

(圖片來源：AWS re:Invent 2016: Move Exabyte-Scale Data Sets with AWS Snowmobile https://youtu.be/8vQmTZTq7nw?t=94)

## 其他功能

### 安全性

Cloud Storage 也可以透過 Cloud IAM 來建立 ACL (Access Control List)，並且透過 Signed URL 來保障資料的安全性。

除此之外，使用者也可以透過 Customer-supplied encryption key (CSEK)，透過自己的加密金鑰對資料進行加密，讓 Google 端完全沒有解密資料的可能性。

### 生命週期

Cloud Storage 可以自行設定自動刪除、自動備份的日期與規則，避免浪費與保障安全。

### 檔案版本

可以設定當同名稱的檔案存取進來後，保留舊版的檔案。避免各種手殘與意外的發生，可以快速地還原先前的版本。不過如同生命週期一樣，檔案版本也需要設定自動保留與刪除的規則，不然檔案可能會越來越多，越來越大，浪費越來越多的錢QQ。
