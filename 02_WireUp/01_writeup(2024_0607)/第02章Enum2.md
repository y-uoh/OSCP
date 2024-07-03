## Enum1で発見したWebサーバにおいてWebページに存在する脆弱性を調査するために、まずはアクセスできるすべてのページを列挙したい。ログインページと推測できるページのファイル名を解答してください。  
  
  取りあえず`gobuster`かな？  
    
```bash
┌──(kali㉿kali)-[~]
└─$ gobuster dir --url http://172.20.15.28:80 --wordlist /usr/share/wordlists/dirb/common.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.20.15.28:80
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2024/06/06 10:10:46 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/admin.php            (Status: 302) [Size: 0] [--> admin_login.php]
/.htaccess            (Status: 403) [Size: 277]                    
/index.html           (Status: 200) [Size: 94]                     
/server-status        (Status: 403) [Size: 277]                    
/uploads              (Status: 301) [Size: 314] [--> http://172.20.15.28/uploads/]
                                                                                  
===============================================================
2024/06/06 10:10:57 Finished
===============================================================

```
  
`admin.php`...怪しすぎる。  
取りあえずもっかい`gobuster`  
と同時にブラウザで開いてみる  
  
```bash
┌──(kali㉿kali)-[~]
└─$ gobuster dir --url http://172.20.15.28:80/admin.php --wordlist /usr/share/wordlists/dirb/common.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.20.15.28:80/admin.php
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2024/06/06 10:16:42 Starting gobuster in directory enumeration mode
===============================================================
Error: the server returns a status code that matches the provided options for non existing urls. http://172.20.15.28:80/admin.php/dd7cba55-3b2f-4e08-8ebe-6a12b546b865 => 302 (Length: 0). To continue please exclude the status code, the length or use the --wildcard switch
  
```
  
時間ないからエラーの原因は調べてないが、とりあえず`gobuster`ではエラー。  
ただブラウザからは、それらしいのがしっかり返ってきた。  
てことで、URLバー見たら  
```bash
http://172.20.15.28/admin_login.php

```
こんな感じだったので、そのまま提出でクリア  
  
## コマンド  
```bash
gobuster dir --url http://172.20.15.28:80 --wordlist /usr/share/wordlists/dirb/common.txt
gobuster dir --url http://172.20.15.28:80/admin.php --wordlist /usr/share/wordlists/dirb/common.txt

```
  
  ## フラグ
  `admin.php`
    
    


