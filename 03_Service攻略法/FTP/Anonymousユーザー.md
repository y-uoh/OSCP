### 1.Anonymousユーザ
- SMBと同じで`パスワード認証なし`でログインできる`Anonymousユーザ`が存在する。（もちろん設定による）  
  
### 2.Anonymousがいるか探索手法
#### 2-1.`nmap -A`でスキャン
example：`10.10.112.191`に対し`-A`スキャンを実行
```bash
┌──(kali㉿kali)-[~]
└─$ nmap -p21 -A 10.10.112.191
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-09 19:40 JST
Nmap scan report for 10.10.112.191
Host is up (0.24s latency).

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
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             353 Apr 24  2020 PUBLIC_NOTICE.txt
Service Info: Host: Welcome

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 29.81 seconds
```
  
#### 2-2.`nse`の`ftp-anon.nse`でスキャン(おススメ)
exampl：`10.10.112.191`に対し`--script ftp-anon.nse`を実行
```bash
┌──(kali㉿kali)-[~]
└─$ nmap -p21 --script ftp-anon.nse 10.10.112.191
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-09 19:49 JST
Nmap scan report for 10.10.112.191
Host is up (0.24s latency).

PORT   STATE SERVICE
21/tcp open  ftp
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             353 Apr 24  2020 PUBLIC_NOTICE.txt

Nmap done: 1 IP address (1 host up) scanned in 15.75 seconds
```
  
### 2-3.Anonymousでログインを試してみる
- 通常の`ftpログイン`同様に`ftp <IP>`でftpに接続にし`Name`を`Anonymous`ででログインできるか試してみる
- 入れたら`Anonymous`がいるという`威力偵察探索`
```bash
┌──(kali㉿kali)-[~]
└─$ ftp 10.10.112.191                   
Connected to 10.10.112.191.
220 Welcome to the administrator FTP service.
Name (10.10.112.191:kali): anonymous
```
  
### 3.Anonymousログインイメージ
- 通常の`ftpログイン`同様に`ftp <IP>`でftpに接続に行く
- `Name`と聞かれるので、`anonymous`と答える
- `Password`を聞かれるが、何も入力することなくそのまま`Enter`
- `230 Login successful`と返ってきて、プロンプトが`ftp>`になればログイン成功  
  
```bash
┌──(kali㉿kali)-[~]
└─$ ftp 10.10.112.191                   
Connected to 10.10.112.191.
220 Welcome to the administrator FTP service.
Name (10.10.112.191:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 

```

  

