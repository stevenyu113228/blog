---
title: "[Day27] Dev Ops (2021 鐵人賽 – Cloud)"
date: 2021-10-12T19:09:00+08:00
draft: false
url: "/2021/10/12/day27-dev-ops/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

其實這次就已經有一個鐵人賽組別完全是 DevOps 了，如果對於這個領域想要比較深入的了解，可以去看該組大大們的文章。我在這邊只打算輕鬆的介紹一下 DevOps 的概念。DevOps 是 Development 跟 Operations 的組合詞，重點是透過一系列的 pipeline 完成自動化的軟體開發與維運，可以簡單的交付軟體與變更架構。

講到 DevOps，首先一定要提到的就是 Continuous Integration 、 Continuous Delivery (CI/CD) ，持續性整合與部屬。簡單來講，就是將程式碼的 Deploy 流程給自動化。而 CI ，持續整合，主要是透過自動化的 Build 程式，確保不會產生環境版本不一致的問題；程式也可以透過 Unit Test 的方式自動化的檢察功能是否正確。CD 持續部屬，則是自動化的將程式碼給 Deploy 到實際環境中，確保每次的更新都可以很順暢。

以 Google Cloud 為例，我們可以將程式碼統一上傳至 Cloud Source Repositories 進行管理。接著，我們可以使用 Cloud Build 對程式碼進行編譯，或包裝成 Docker 的 Image。透過 Build Trigger 追蹤 Repositories 是否有更改，如果有任何的更動就自動化的觸發 Cloud Build。最終， Build 出來的 Container 可以儲存在 Container Registry 中。

在 DevOps 的環境中，我們也可以透過 Jenkins 等程式進行自動化的建置，Jenkins 也可以在 Google Cloud 的 Marketplace 直接的進行安裝，而管理的過程，我們也可以透過 Google Kubernetes Engine 進行管理。蒐集 Log 的部分，也可以透過 ELK (Elasticsearch、Logstash、Kibana) 進行有效率的資料蒐集與視覺化。

透過 DevOps 我們可以避免很多人為的失誤，因為一系列的功能全部都自動化了，~~所以對於懶人而言也非常的好用！~~
