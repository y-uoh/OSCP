## 対象のサーバにポートスキャンを実行します。TCPのポート番号1~10000のうち、開放されているポートの数を解答してください。  
  
まずは普通に`nmap`  
   
```bash
┌──(kali㉿kali)-[~]
└─$ nmap -p 1-10000 172.20.15.28
Starting Nmap 7.92 ( https://nmap.org ) at 2024-06-06 10:06 JST
Nmap scan report for 172.20.15.28
Host is up (0.015s latency).
Not shown: 9995 closed tcp ports (conn-refused)
PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Nmap done: 1 IP address (1 host up) scanned in 16.04 seconds

```  
  
### コマンド 
-> `nmap -p 1-10000 172.20.15.28`   
###flag 
-> `5`
