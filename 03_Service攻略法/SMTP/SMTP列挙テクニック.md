## 1.SMTPでの列挙
設定が不十分なメールサーバは、ターゲットへの最初の足掛かりになることがよくある。  
例えばいつものように、メールサーバのバージョンなどの脆弱性を突いたexploitによる侵入だったり、サーバーのコマンドを活用してユーザを列挙し、その後hydraなどのツールを用いてSSHでログインするなどである。  
ここでは主に`サーバーの列挙手法`と`ユーザの列挙手法`の２つのテクニックについて解説する。  
  
## 2.サーバーの列挙手法
サーバーの列挙手法を２つ取り上げる
- `nmap -A`による列挙（推奨）
- `Metasploitのsmtp_version`モジュールによる列挙
  
今後、Metasploitが使えない環境があるかもしれないので、Metasploitを使わない手法を積極的に押さえる  
  
### 2-1.`nmap -A`による列挙（推奨）
ベターな手法なので下記で実行例を示す  
```bash
┌──(kali㉿kali)-[~]
└─$ nmap -T4 -A 10.10.227.12                                                           
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-12 15:02 JST
Nmap scan report for 10.10.227.12
Host is up (0.21s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 62:a7:03:13:39:08:5a:07:80:1a:e5:27:ee:9b:22:5d (RSA)
|   256 89:d0:40:92:15:09:39:70:17:6e:c5:de:5b:59:ee:cb (ECDSA)
|_  256 56:7c:d0:c4:95:2b:77:dd:53:d6:e6:73:99:24:f6:86 (ED25519)
25/tcp open  smtp    Postfix smtpd
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=polosmtp
| Subject Alternative Name: DNS:polosmtp
| Not valid before: 2020-04-22T18:38:06
|_Not valid after:  2030-04-20T18:38:06
|_smtp-commands: polosmtp.home, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8
Service Info: Host:  polosmtp.home; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 44.02 seconds
```
詳細を見ていく  
```bash
25/tcp open  smtp    Postfix smtpd
```
smtpのサービス名が判明：`smtp    Postfix smtpd`
  
```bash
|_smtp-commands: polosmtp.home, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8
```
このSMTPで使用できるコマンドが判明：`VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8`  
```bash
Service Info: Host:  polosmtp.home; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
ドメイン名が判明 ：`Info: Host:  polosmtp.home`

### 2-2.`Metasploitのsmtp_version`モジュールによる列挙
ここも実行例を用いて解説  
```bash
└─$ msfconsole -q
msf6 > search smtp_version

Matching Modules
================

   #  Name                                 Disclosure Date  Rank    Check  Description
   -  ----                                 ---------------  ----    -----  -----------
   0  auxiliary/scanner/smtp/smtp_version                   normal  No     SMTP Banner Grabber


Interact with a module by name or index. For example info 0, use 0 or use auxiliary/scanner/smtp/smtp_version

msf6 > use 0
```
Metasploitのsearchでsmtp_versionで検索するとでてくるので、useで指定する  
```bash
msf6 > use 0
msf6 auxiliary(scanner/smtp/smtp_version) > options 

Module options (auxiliary/scanner/smtp/smtp_version):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOSTS                    yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/ba
                                       sics/using-metasploit.html
   RPORT    25               yes       The target port (TCP)
   THREADS  1                yes       The number of concurrent threads (max one per host)


View the full module info with the info, or info -d command.

msf6 auxiliary(scanner/smtp/smtp_version) > set RHOSTS 10.10.227.12
RHOSTS => 10.10.227.12
```
`options`をみるとRHOSTSがRequiredのyesになっているので、setで設定する
```bash
msf6 auxiliary(scanner/smtp/smtp_version) > run

[+] 10.10.227.12:25       - 10.10.227.12:25 SMTP 220 polosmtp.home ESMTP Postfix (Ubuntu)\x0d\x0a
[*] 10.10.227.12:25       - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed

```
`run`で走らせて、結果をみる  
`[+] 10.10.227.12:25       - 10.10.227.12:25 SMTP 220 polosmtp.home ESMTP Postfix (Ubuntu)\x0d\x0a`  
↑を部分から、ドメイン：`polosmtp.home`、サービス名：`Postfix`であることが判明  
  
## 3.ユーザの列挙手法
SMTPでのユーザの列挙手法の基本的な考え方としては、`SMTPのコマンドを使用してユーザ名を吐き出す手法`ところにある。  
なので押さえておくべきポイントは、`ユーザ名を吐き出させるSTMPのコマンド`と`SMTPのコマンドを使用する方法`についての２点である。  
  
### 3-1.ユーザ名を吐き出させるSTMPのコマンド
#### VRFYコマンド
- 特定のユーザのメールボックスの出力
- 使い方：`VRFY <ユーザ名>`
  
#### EXPNコマンド
- 特定のメーリングリストの出力
- 使い方：`EXPN <ユーザ名 or メーリングリスト名>`
  
### 3-2.SMTPのコマンドを使用する方法
そもそもこれがどういうことかというと、SMTPのコマンドを実行するには、当然サーバー内に侵入していなければ行うことが出来ない。なのでそれをどうやって外部にいる状態からコマンドを実行するかという問題にぶち当たったときにとる手法を解説するということである。  
仮に既に`Telnet`などでサーバー内に入れるのであれば、それで侵入し上記のコマンドを手動で実行して有効なユーザを取得することは可能である。ただどちらのコマンドも当初の`ユーザー名`がわからなければ、例えば`root`や`administrator`など当てずっぽうで試行していくほかない。それだとあまりにも非効率なので、以下２つのツールを使用してユーザを列挙していく。  
- `smtp-user-enum`の使用（推奨）
- Metasploitの`smtp_enum`モジュールの使用  
  
#### `smtp-user-enum`の使用（推奨）
- ターゲットのSMTPに対し、コマンドを実行できるツール
- ユーザ名にWordlistを使用すれば、リスト攻撃的にリストに記載されている全てのユーザでコマンドが実行できる  
- つまり、ユーザ名のWordlistを指定してVRFYコマンドを実行すれば、有効なユーザが列挙できるということである。  

```bash
基本構文：
smtp-user-enum -M <コマンド> -D <ドメイン名> -t <IP> -U <wordlist>
```
色々オプションはあるが、必要そうなものだけ列挙  
- `-M`：実行したいコマンドを指定。必須
- `-D`：ドメイン名の指定。最悪なくてもいける
- `-t`：ターゲットIPの指定。必須
- `-U`：ユーザ名のWordlistを指定。このWordlistの網羅性がユーザ名列挙の精度に関わる
- `-h`：ヘルプオプション。細かく見たいときはこれ出力して、DeepL  
  
```bash
実行例：
┌──(kali㉿kali)-[~]
└─$ smtp-user-enum -M VRFY -t 10.10.227.12 -U /usr/share/wordlists/seclists/Usernames/top-usernames-shortlist.txt 
Starting smtp-user-enum v1.2 ( http://pentestmonkey.net/tools/smtp-user-enum )

 ----------------------------------------------------------
|                   Scan Information                       |
 ----------------------------------------------------------

Mode ..................... VRFY
Worker Processes ......... 5
Usernames file ........... /usr/share/wordlists/seclists/Usernames/top-usernames-shortlist.txt
Target count ............. 1
Username count ........... 17
Target TCP port .......... 25
Query timeout ............ 5 secs
Target domain ............ 

######## Scan started at Wed Jun 12 16:09:27 2024 #########
10.10.227.12: root exists
10.10.227.12: administrator exists
10.10.227.12: vagrant exists
######## Scan completed at Wed Jun 12 16:09:30 2024 #########
3 results.

17 queries in 3 seconds (5.7 queries / sec)
```
ユーザ列挙の一例  
ユーザー名のWordlistを使用してVRFYコマンドを実行しているので、これで返ってきたユーザはVRFYコマンドが成功したユーザ、つまり有効なユーザ名ということである。  
```bash
######## Scan started at Wed Jun 12 16:09:27 2024 #########
10.10.227.12: root exists
10.10.227.12: administrator exists
10.10.227.12: vagrant exists
```
ここの部分が有効なユーザということである。  
  
#### Metasploitの`smtp_enum`モジュールの使用
これは実行例を用いて解説  
```bash
msf6  > search smtp_enum

Matching Modules
================

   #  Name                              Disclosure Date  Rank    Check  Description
   -  ----                              ---------------  ----    -----  -----------
   0  auxiliary/scanner/smtp/smtp_enum                   normal  No     SMTP User Enumeration Utility


Interact with a module by name or index. For example info 0, use 0 or use auxiliary/scanner/smtp/smtp_enum

msf6 auxiliary(scanner/smtp/smtp_version) > use 0
```
searchでsmtp_enumと検索し、モジュールを指定  
  
```bash
msf6 auxiliary(scanner/smtp/smtp_enum) > options 

Module options (auxiliary/scanner/smtp/smtp_enum):

   Name       Current Setting                   Required  Description
   ----       ---------------                   --------  -----------
   RHOSTS                                       yes       The target host(s), see https://docs.metasploit.com/docs/
                                                          using-metasploit/basics/using-metasploit.html
   RPORT      25                                yes       The target port (TCP)
   THREADS    1                                 yes       The number of concurrent threads (max one per host)
   UNIXONLY   true                              yes       Skip Microsoft bannered servers when testing unix users
   USER_FILE  /usr/share/metasploit-framework/  yes       The file that contains a list of probable users accounts.
              data/wordlists/unix_users.txt


View the full module info with the info, or info -d command.

msf6 auxiliary(scanner/smtp/smtp_enum) > set RHOSTS 10.10.227.12
RHOSTS => 10.10.227.12
```
optionsでRequiredがyesになっているRHOSTSにターゲットIPを設定する
  
```bash
msf6 auxiliary(scanner/smtp/smtp_enum) > set USER_FILE /usr/share/wordlists/seclists/Usernames/top-usernames-shortlist.txt
USER_FILE => /usr/share/wordlists/seclists/Usernames/top-usernames-shortlist.txt
```
optionsのUSER_FILEにはデフォルトで既にWordlistが設定されているが、量が多く実行が遅いため、/usr/share/wordlists/seclists/Usernames/top-usernames-shortlist.txtを設定しなおす。  
なお、SeclistsというWordlistのパッケージみたいなやつは推奨なのでインストールしておくのがよい
  
```bash
msf6 auxiliary(scanner/smtp/smtp_enum) > run

[*] 10.10.227.12:25       - 10.10.227.12:25 Banner: 220 polosmtp.home ESMTP Postfix (Ubuntu)
[+] 10.10.227.12:25       - 10.10.227.12:25 Users found: administrator
[*] 10.10.227.12:25       - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```
administratorという有効なユーザが存在することが出力されている。  
このモジュールはsmtp-user-enumのようにWordlistに乗っている有効なユーザを全て出力してくれない。おそらく一番初めに見つかったユーザのみ出力する。  


