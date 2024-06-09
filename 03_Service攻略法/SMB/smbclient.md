## smbclient使い方
### 1.SMB共有にアクセス
#### 1-1.基本構文
```bash
smbclient //<ip>/<共有フォルダ名>
```
#### 1-2.ユーザ指定、ポート指定
```bash
smbclient //<ip>/<共有フォルダ名> -U <ユーザ名> -p <ポート番号>
```
- `-U`：ユーザの指定
- `-p`：ポートの指定  
  
### 2.SMB操作
#### 2-1.ファイルのダウンロード
```bash
get <ファイル名>
```
- `-get`コマンドでダウンロード
- ダウンロード先は`接続時のカレントディレクトリ`
- ファイル名にスペースがあるときは文字列を`"(ダブルクォーテーション)`で囲む。以下参考  
```bash
smb: \> ls
  .                                   D        0  Tue Apr 21 20:08:23 2020
  ..                                  D        0  Tue Apr 21 19:49:56 2020
  .cache                             DH        0  Tue Apr 21 20:08:23 2020
  .profile                            H      807  Tue Apr 21 20:08:23 2020
  .sudo_as_admin_successful           H        0  Tue Apr 21 20:08:23 2020
  .bash_logout                        H      220  Tue Apr 21 20:08:23 2020
  .viminfo                            H      947  Tue Apr 21 20:08:23 2020
  Working From Home Information.txt      N      358  Tue Apr 21 20:08:23 2020
  .ssh                               DH        0  Tue Apr 21 20:08:23 2020
  .bashrc                             H     3771  Tue Apr 21 20:08:23 2020
  .gnupg                             DH        0  Tue Apr 21 20:08:23 2020

		12316808 blocks of size 1024. 7567324 blocks available
```
共有フォルダをみると`Working From Home Information.txt`があるが、ファイル名に謎のスペースがいっぱい空いてる  
そうゆうときは↓    
```bash
smb: \> get "Working From Home Information.txt"

getting file \Working From Home Information.txt of size 358 as Working From Home Information.txt (0.4 KiloBytes/sec) (average 0.4 KiloBytes/sec)
```
`smb: \> get "Working From Home Information.txt"`のような感じで"（ダブルクォーテーション）で文字列を囲めばよい  
  
#### 2-2.ディレクトリ移動
```bash
cd <移動先ディレクトリ名>
```
- `cd`はLinuxと同じコマンドが使える
  
#### 2-3.ディレクトリを開く
```bash
ls <ディレクトリ>
dir <ディレクトリ>
```
- `ls`も`dir`もどちらも使える
  
#### 2-4.使えるコマンド一覧
```bash
smb: \> help
```
- `help`コマンドで使えるコマンドの一覧を出力できる。
- 以下、実行例
```bash
smb: \> help
?              allinfo        altname        archive        backup         
blocksize      cancel         case_sensitive cd             chmod          
chown          close          del            deltree        dir            
du             echo           exit           get            getfacl        
geteas         hardlink       help           history        iosize         
lcd            link           lock           lowercase      ls             
l              mask           md             mget           mkdir          
more           mput           newer          notify         open           
posix          posix_encrypt  posix_open     posix_mkdir    posix_rmdir    
posix_unlink   posix_whoami   print          prompt         put            
pwd            q              queue          quit           readlink       
rd             recurse        reget          rename         reput          
rm             rmdir          showacls       setea          setmode        
scopy          stat           symlink        tar            tarmode        
timeout        translate      unlock         volume         vuid           
wdel           logon          listconnect    showconnect    tcon           
tdis           tid            utimes         logoff         ..             
!              
```        
  
#### 2-4.smb操作備忘録
`windows`の仕様に近いからか、`cmdコマンド`は使える印象  
しかし、ファイルを出力する`type`はもちろん`cat`も使えなかったので、ファイルの中身を見るときは一度`get`でローカルに持ってきてから開くしかない。  
どのコマンドが使えるは、その都度確認する必要がある。  
