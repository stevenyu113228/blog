# 發文指南

## 新增文章

1. 複製模板建立新檔案：

```bash
cp content/posts/_template.md content/posts/your-slug.md
```

2. 編輯 front matter：

```yaml
---
title: "文章標題"
date: 2026-03-01T20:00:00+08:00    # 發文時間（+08:00 台灣時區）
draft: false                         # true = 草稿不發布，false = 發布
url: "/2026/03/01/your-slug/"        # 必須手動填，格式 /:year/:month/:day/:slug/
categories:                          # 至少選一個分類
  - 資訊安全
showToc: true                        # 顯示目錄
TocOpen: false                       # 目錄預設收合
---
```

3. 用 Markdown 撰寫內容

## 分類列表

現有分類（可直接使用，也可新增）：

| 分類 | 說明 |
|------|------|
| 資訊安全 | 上層分類 |
| 網頁安全 | Web Security |
| 滲透測試 | PT / OSCP 相關 |
| Writeup | CTF / HTB / THM |
| iTHome 鐵人賽 | 鐵人賽系列文 |
| Cloud | GCP 等雲端 |
| Block Chain | 區塊鏈 |
| 廢文 | 雜談 |
| 小玩具 | 工具 / Side Project |

## 圖片

把圖片放到 `static/uploads/` 下，依年月分目錄：

```
static/uploads/2026/03/screenshot.png
```

在文章中引用：

```markdown
![說明文字](/uploads/2026/03/screenshot.png)
```

## 程式碼

用 fenced code block 加語言標示：

````markdown
```python
print("hello")
```
````

支援的語言：python, bash, go, javascript, php, sql, yaml, json, html, css 等。

## 本地預覽

```bash
hugo server
```

開啟 http://localhost:1313/ 預覽，存檔後自動 reload。

加 `-D` 可預覽草稿：

```bash
hugo server -D
```

## 發布

```bash
git add content/posts/your-slug.md
git add static/uploads/2026/03/   # 如果有新圖片
git commit -m "post: 文章標題"
git push
```

推上 GitHub 後，GitHub Actions 會自動 build 並部署到 blog.stevenyu.tw。

## URL 規則

- 文章：`/2026/03/01/your-slug/`（必須在 front matter 的 `url` 手動設定）
- 分類：`/categories/分類名/`
- 搜尋：`/search/`
- 歸檔：`/archives/`
- 關於：`/about/`

## 注意事項

- `url` 欄位必填，Hugo 靠這個產生正確路徑
- 中文 slug 可以用，例如 `url: "/2026/03/01/我的文章/"`
- `draft: true` 的文章不會被 build，確認要發布才改成 `false`
- 模板檔 `_template.md` 因為 `draft: true` 所以不會出現在網站上
