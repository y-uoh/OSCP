## 1.ログイン操作
### 1-1.基　本
```bash
ftp <IP>
```
-`ftp`の後に、`サーバーのIP`を入力
- のち`Name`として、ユーザ名を聞かれるので、`任意のユーザ名`、あるいは`Anonumous`と入力
- `Password`を求められるので、`パスワード`を入力。なお`Anonymousユーザ`でのログインの場合、何も入力せずに`Enter`
- `230 Login successful.`と出力され、プロンプトが`ftp>`で戻ってくればログイン成功
  
### 1-2.example
- `10.10.119.98`のFTPサーバに接続
- `Name`に`mike`
- `Password`を入力
- ログイン成功
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
ftp> 
```
  
## 2.FTP操作  
基本は、`SMB`と同じようなコマンドが使える。
  
### 2-1.データのダウンロード
```bash
ftp>get <ファイル名>
```
- `get`コマンドでファイルのダウンロード
- 接続時の`カレントディレクトリ`にダウンロードされる
  
#### example
- `ftp.txt`をダウンロード
- `226 Transfer complete.`の表示でダウンロード成功
```bash
ftp> get ftp.txt
local: ftp.txt remote: ftp.txt
229 Entering Extended Passive Mode (|||11876|)
150 Opening BINARY mode data connection for ftp.txt (26 bytes).
100% |***********************************|    26      362.72 KiB/s    00:00 ETA
226 Transfer complete.
26 bytes received in 00:00 (0.07 KiB/s)
```  
  
### 2-2.データのアップロード
```bash
ftp>send <ファイル名>
```
- `send`コマンドでファイルのアップロード
   
#### example
- `test.txt`をアップロード
- `Transfer complete.`でアップロード成功
```bash
ftp> send test.txt
local: test.txt remote: test.txt
229 Entering Extended Passive Mode (|||7635|)
150 Ok to send data.
     0        0.00 KiB/s 
226 Transfer complete.
``` 
  
### 2-3.その他一覧
- cd：ディレクトリ移動
- ls：ディレクトリ表示（dirも可）

### 2-4.ヘルプ
```bash
 ftp>help
 ```  
 - `help`コマンドで使えるコマンドの一覧を表示
   
#### ヘルプ一覧
```bash
ftp> help
Commands may be abbreviated.  Commands are:

!		epsv6		mget		preserve	sendport
$		exit		mkdir		progress	set
account		features	mls		prompt		site
append		fget		mlsd		proxy		size
ascii		form		mlst		put		sndbuf
bell		ftp		mode		pwd		status
binary		gate		modtime		quit		struct
bye		get		more		quote		sunique
case		glob		mput		rate		system
cd		hash		mreget		rcvbuf		tenex
cdup		help		msend		recv		throttle
chmod		idle		newer		reget		trace
close		image		nlist		remopts		type
cr		lcd		nmap		rename		umask
debug		less		ntrans		reset		unset
delete		lpage		open		restart		usage
dir		lpwd		page		rhelp		user
disconnect	ls		passive		rmdir		verbose
edit		macdef		pdir		rstatus		xferbuf
epsv		mdelete		pls		runique		?
epsv4		mdir		pmlsd		send
ftp> 
```
  