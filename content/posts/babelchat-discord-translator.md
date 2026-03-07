---
title: "BabelChat 基於 AI 的 Discord 翻譯軟體"
date: 2026-03-07T21:00:00+08:00
draft: true
url: "/2026/03/07/babelchat-discord-translator/"
categories:
  - 小玩具
showToc: true
TocOpen: false
---

**TL;DR**：用 Claude Code Vibe Coding 了一個 Discord 翻譯插件，基於 Vencord，支援任何 OpenAI 相容 API，可以一鍵翻譯整串對話、自動翻譯新訊息、打字時先翻譯再送出。免費、開源、附一鍵安裝腳本。[GitHub](https://github.com/stevenyu113228/BabelChat)

---

現在無論資安、WEB 3 相關的社群都流行在 Discord 上開啟相關的 Server。因此我的 Discord 上常常充斥著多種不同的語言，每次遇到看不懂或懶得看的時候，我不是打開 Google 翻譯 / DeepL 就是打開 ChatGPT / Gemini，每天重複很多次，多到乾脆懶得看ㄌ。

網路上現在當然也有一些 Solution，例如基於 Better Discord 的 Translator Plugin 等，不過這些翻譯仍然是基於 Google 翻譯等 API，而且使用操作起來並沒有想像中直覺，沒辦法大規模的翻譯一整串的聊天訊息。或是直接安裝翻譯 Bot，但這又會驚動其他使用者，畢竟對我來說，潛水偷看才是最常發生的事 XD。

首先我們需要知道一下 Discord 預設其實是不允許安裝 Extension 的，社群為了解決這個問題，最簡單的方法當然就是使用瀏覽器，瀏覽器的翻譯外掛就有非常多的選擇，例如我最常用的 沈浸式翻譯 (Immersive Translate)，不過非原生用起來總是沒有很舒服，所以我希望使用盡可能原生的方法在 Discord 中解決這個問題。另外一個我的需求是希望可以使用 AI 輔助翻譯，出現原文跟中文的對照，實現類似 沈浸式翻譯 的體驗。

## Vencord

因此如果想要對原生的 Discord 動手腳的話，最常使用的工具叫做 Vencord，這邊需要知道 Discord 本質上是一個 Electron 的 APP，也就是包了一個 Chromium 在裡面跑 React，所以只要有辦法 Inject JS 的話就可以直接增加功能。不過直接改 Discord 的程式碼是一件很痛苦的事情，它經過了一系列 webpack 的打包壓縮等，每一次更新都需要重新逆向一次。

因此 Vencord 就解決了這個問題，[Vencord](https://github.com/Vendicated/Vencord) 是目前最活躍的 Discord 客戶端 Mod 框架，他直接提供多種的抽象層，使我們可以直接 Call Vencord 提供的 API 來擴充他們。它也提供 Native Module 的支援，允許在裡面跑 Node.js 不受瀏覽器 Sandbox 的限制。

## Vibe Coding

在 Vibe Coding 的時代，遇到這種需求當然就直接請 Claude Code 出場囉，總之把這些需求跟 Claude Code 說一說，使用 Plan Mode 搭配 [SuperPowers](https://github.com/obra/superpowers) Skills 大概花了半小時就開發完畢。

Vencord 的插件分兩種，官方套件以及 Userplugin，由於這是一個自己開發的套件，而且會需要依賴第三方的 LLM API，所以我這邊暫時不打算送審官方的套件庫，而使用 Userplugin 則需要從原始碼開始 Build，所以我就再 Vibe 了一個一鍵安裝的腳本。

## 功能介紹

最終成品長這樣：

![Demo](https://raw.githubusercontent.com/stevenyu113228/BabelChat/main/img/demo.png)

BabelChat 支援以下幾種翻譯方式：

- **右鍵翻譯 / 懸停翻譯**：對單則訊息按右鍵選 "BabelChat Translate"，或是滑鼠懸停時點翻譯圖示，翻譯結果會直接顯示在原始訊息下方。
- **自動翻譯**：設定裡開啟 Auto Translate，之後所有新訊息（包含自己發的）都會自動翻譯。適合全程潛水偷看的時候用。
- **批量翻譯**：聊天區右邊有一個浮動按鈕，點一下就會把畫面上所有可見的訊息一次翻譯完。進到一個新頻道想快速了解在聊什麼的時候超方便。
- **撰寫翻譯**：在輸入框打完母語後，點聊天欄的翻譯按鈕，輸入框的內容會被替換成翻譯後的文字，確認沒問題再送出。不用再另外開翻譯工具。

API 的部分支援任何 OpenAI 相容的端點，所以 OpenAI、Gemini、Ollama、LM Studio 都可以用。如果不想花錢的話，推薦用 [OpenRouter](https://openrouter.ai/) 的免費模型，不用信用卡就能註冊。

## 安裝教學

### 一鍵安裝

1. 到 [GitHub](https://github.com/stevenyu113228/BabelChat) 點 **Code > Download ZIP**（或用 `git clone`），解壓縮。
2. 打開終端機（macOS: `Cmd + 空白鍵` 搜尋 Terminal / Windows: `Win + X` 選 PowerShell）。
3. 執行安裝腳本：

```bash
cd ~/Downloads/BabelChat    # 改成你的路徑
bash install.sh             # macOS / Linux
.\install.ps1               # Windows
```

腳本會自動處理 Vencord 下載、套件安裝、編譯、注入，全部一條龍。注入前記得**先關閉 Discord**，注入器問你版本的時候**選 Discord（Stable）**就好。

4. 重開 Discord → 設定 → Vencord → Plugins → 搜尋 **BabelChat** → 啟用。

### 設定 API

啟用後點 BabelChat 的設定齒輪，填入 API 資訊。最簡單的免費方案：

1. 到 [openrouter.ai](https://openrouter.ai/) 註冊（Google 一鍵登入）。
2. 到 [API Keys](https://openrouter.ai/keys) 建立金鑰。
3. 填入設定：

| 欄位 | 值 |
|---|---|
| API Endpoint | `https://openrouter.ai/api/v1/chat/completions` |
| API Key | 你的 OpenRouter 金鑰 |
| Model | `google/gemma-3-27b-it:free` |
| Target Language | 你想翻譯成的語言 |

其他免費模型還有 `meta-llama/llama-3.3-70b-instruct:free`、`mistralai/mistral-small-3.1-24b-instruct:free` 等，可以到 [OpenRouter Models](https://openrouter.ai/models?q=free) 看完整列表。

## 結語

總之如果你也是常在各國不同 Discord 群組潛水的人，歡迎來玩玩看：

**GitHub**：[github.com/stevenyu113228/BabelChat](https://github.com/stevenyu113228/BabelChat)
