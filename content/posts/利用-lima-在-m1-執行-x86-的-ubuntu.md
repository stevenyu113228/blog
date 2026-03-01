---
title: 利用 Lima 在 M1 執行 x86 的 Ubuntu
date: 2022-08-21T21:34:04+08:00
draft: false
url: "/2022/08/21/利用-lima-在-m1-執行-x86-的-ubuntu/"
categories:
  - 小玩具
showToc: true
TocOpen: false
---

超簡單就可以直接在 M1 上有類似 WSL 體驗的 x86_64 Ubuntu ， Lima 是透過 QEMU 進行執行的，所以嚴格來說他是一種虛擬機，不太像 WSL 是微軟大大的黑魔法。

## 安裝

```bash
brew install lima
```

準備一個 Yaml，這邊我是直接抄官方的 [Example](https://raw.githubusercontent.com/lima-vm/lima/master/examples/ubuntu.yaml)，並修改，增加第一行的 arch，根據官方文件表示，增加 arch 就能直接指定 CPU 型態

```yaml
arch: "x86_64"

images:
# Try to use release-yyyyMMdd image if available. Note that release-yyyyMMdd will be removed after several months.
- location: "https://cloud-images.ubuntu.com/releases/22.04/release-20220712/ubuntu-22.04-server-cloudimg-amd64.img"
  arch: "x86_64"
  digest: "sha256:86481acb9dbd62e3e93b49eb19a40c66c8aa07f07eff10af20ddf355a317e29f"
- location: "https://cloud-images.ubuntu.com/releases/22.04/release-20220712/ubuntu-22.04-server-cloudimg-arm64.img"
  arch: "aarch64"
  digest: "sha256:e1ce033239f0038dca5ef09e582762ba0d0dfdedc1d329bc51bb0e9f5057af9d"
# Fallback to the latest release image.
# Hint: run `limactl prune` to invalidate the cache
- location: "https://cloud-images.ubuntu.com/releases/22.04/release/ubuntu-22.04-server-cloudimg-amd64.img"
  arch: "x86_64"
- location: "https://cloud-images.ubuntu.com/releases/22.04/release/ubuntu-22.04-server-cloudimg-arm64.img"
  arch: "aarch64"

mounts:
- location: "~"
- location: "/tmp/lima"
  writable: true
```

## Build 環境

這邊的 ./ubuntu.yaml 就是前面準備的內容，而 --name 則是帶之後要給他的名字

```
limactl start ./ubuntu.yaml --name ubuntu-amd64
```

並選擇 "Proceed with the current configuration"

## 使用

```bash
limactl shell ubuntu-amd64  # 進入 Shell
limactl stop ubuntu-amd64 # 關閉機器
limactl delete ubuntu-amd64 # 刪除機器
```

![](/uploads/2022/08/截圖-2022-08-21-下午9.32.48-1024x163.png)
