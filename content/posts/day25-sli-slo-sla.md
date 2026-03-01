---
title: "[Day25] SLI , SLO , SLA (2021 鐵人賽 – Cloud)"
date: 2021-10-10T19:08:00+08:00
draft: false
url: "/2021/10/10/day25-sli-slo-sla/"
categories:
  - iTHome 鐵人賽
  - "真ㄉ有可能在 30 天內搞懂 Cloud ㄇ ?__?"
showToc: true
TocOpen: false
---

今天來介紹雲端的管理，常常出現的三個名詞，在先前的文章中，我應該也有使用過了一部分。這三個名詞長的很像，而背後的意義卻有滿多的差異，都算是滿重要，滿常用的的名詞哦！這些算是服務級別的術語，描述服務常見需要衡量的指標。

## Service Level Indicators , SLI

SLI 被稱為服務級別指標，我們可以理解成它是我們需要衡量一個服務的哪些指標。例如一個服務的網路吞吐量，延遲時間與錯誤率等。在 SLI 中，我們只需要訂出，我們需要衡量哪些東西就好，而剩下的則是交由 SLO 進行規範。

## Service Level Objectives , SLO

SLO 被稱為服務級別目標，通常是提供服務者需要明確定義給客戶的量化指標。這邊我們就可以定義 SLI 必須要符合的規定 Range，例如我們要求一個網頁在台灣地區的回應時間必須小於 10 毫秒，等明確的定義。

## Service Level Agreements , SLA

SLA 則是服務的級別協定，通常是服務商與客戶之間正式需要簽的合約，保障系統的穩定性。而服務商如果沒有達到指定的 SLA 則可以被客戶要求賠償，而 SLA 會涵蓋的內容相對而言最具有限制性。

我們以 [Google Cloud Storage SLA](https://cloud.google.com/storage/sla) 網頁上提到的，來觀察它的 SLI 、 SLO 以及 SLA 分別是什麼。例如 Standard storage 的 Monthly Uptime Percentage >= 99.95，這就算是一個 SLA 的規定。而它們的也有在下文題到，如果沒有達到指定的 SLO 會給予賠償等。
