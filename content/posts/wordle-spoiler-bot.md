---
title: Wordle Spoiler Bot
date: 2022-02-16T20:39:00+08:00
draft: false
url: "/2022/02/16/wordle-spoiler-bot/"
categories:
  - 廢文
showToc: true
TocOpen: false
---

GitHub : [https://github.com/stevenyu113228/Wordle-Spoiler-Bot](https://github.com/stevenyu113228/Wordle-Spoiler-Bot)

感覺 Wordle 也紅一陣子了，但我覺得最近的題目都好難，都解不出來 QQ，一氣之下就寫了一個 Telegram 的腳本來自動地告訴我當天的答案。

## 原始碼分析

觀察 [https://www.nytimes.com/games/wordle/index.html](https://www.nytimes.com/games/wordle/index.html) 首頁的原始碼，可以看到一個重點的 JavaScript，main.bfba912f.js (後面的 Hash 可能會變)

按下去後會取得一大串壓縮後的 JS，可以先把它送到 JS Beautifier 上面，就能看到大量的解答ㄌ，題目是每天照順序取一個出來。

![](/uploads/2022/03/截圖-2022-03-02-下午8.29.52-1024x640.png)

理論上我們可以把這個 List 取出來之後，透過計算 Offset 的方法就能取得當天以及未來的答案。

但這樣的問題是，如果 Wordle 未來修改了原始碼，可能就會預測失敗 QQ。

因此我的解法是大致的看完原始碼後，發現他會呼叫一個 Class 跟他的 Function

```javascript
new wordle.bundle.GameApp()
```

這個產出的 Object 中，有一個屬性叫做 `solution` 即為答案。

所以我們可以在 F12 中輸入下列指令取得答案

```javascript
(new wordle.bundle.GameApp()).solution
```

接下來再把這個功能給用 Selenium 包起來，透過 Selenium 可以執行 js 的特性來取得答案

```python
driver.execute_script("return (new wordle.bundle.GameApp()).solution")
```

最後調整時區、套上 Telegram 的 API 、 包成 Docker 就大功告成了！

![](/uploads/2022/03/截圖-2022-03-02-下午8.40.23-652x1024.png)
