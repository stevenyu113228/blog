---
title: "[Day16] Functions (2021 鐵人賽 – Cloud)"
date: 2021-10-01T19:01:00+08:00
draft: false
url: "/2021/10/01/day16-functions/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

Cloud Function 是一款 Serverless 的服務，使用者不需要管理伺服器。Google 官方說，它是一種 FaaS (Function as a Services，函式即服務)，它可以透過透過 HTTP 的 API 來基於事件做出相對應的動作。在 AWS 上，最相似的功能是 Lambda。

目前 Cloud Functions 是透過 Node.js 的 JavaScript 進行編寫，並運行在 GCP 中。

Cloud Functions 最主要可以區分成兩個重點 Event (事件) 與 Trigger (觸發)，Event 可以是 HTTP 的 Webhook，Cloud Storage、Cloud Pub/Sub 、 Cloud FireStore 等方式，我們可以收取、訂閱指定的 Event，並設定當 Event 發生後，需要做的事情 (Trigger) 是什麼。

舉例來說，我們如果希望透過 Cloud Storage 建立一個線上的相簿，而相簿通常除了完整尺寸的原圖之外，為了減緩頻寬、加快載入速度，通常會採用縮圖預覽的格式。我們就可以透過 Cloud Functions 來建立一個 Event ，設計為 「當 Cloud Storage 上傳了新的圖片」，則觸發 Trigger 「透過 Function 自動產出一張壓縮尺寸後的縮圖，存放進 Cloud Storage 中」。

除了收送 Storage 的內容之外，也可以藉由 Webhook 整合其他的服務，例如串接 Cloud Vision API 做影像辨識；自動推播訊息，做為行動裝置的後端等。

FaaS 最大的特性是 無狀態 (Stateless)，每一次觸發腳本的請求都是獨立、互相不會干擾的，以前面的例子為例，這個腳本就只負責做當前這張縮小圖片的功能，並回傳到 Storage 中，不會去考慮先前儲存的狀態。

透過 FaaS 可以提高開發的效率，專注於開發單一的功能，並只在需要用到該功能時付錢即可。但 FaaS 最大的缺點是，容易產生程式功能的碎片化，進而導致不方便統整的管理。
