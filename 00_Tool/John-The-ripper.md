## 概　要
- パスワード解析ツール

## 書　式
```bash
john <オプション> <解析モード> <解析対象ファイル>
```
### 説　明  
### オプション
- `--format=<ハッシュアルゴリズム等>`
  - 解析対象ファイルのハッシュアルゴリズムを指定することが可能
  - 通常、パスワードファイルから自動的に判断し、適切な方法で解析
  - 解析対象ファイルが`/etc/passwd`ファイルのような形式の場合、`--format=cypto`といった指定の仕方もあるので、必要に応じて確認

### 解析モード
- `--wordlist=<ワードリストファイル>`
  - `Wordlist`から解析するモード
  - 一番効率よく一番よく使う
  - `Wordlist`は`/usr/share/wordlists/`の中にある
  - `Wordlist`の頻出は、`/usr/share/wordlists/rockyou.txt`である
- `--single`
  - パスワードがなかったり、ユーザ名＝パスワードだったり、ユーザ名を少し並べ替えるくらいの、簡単なパスワードのみ解析するモード
  - singleモードで解析されるパスワードはかなり脆弱なパスワードが設定されているといえる
- `--incremental=<使用文字>`
  - aから順番にb,c,d...aaaaaaなどと見つかるまでパスワードを１文字ずつ変更していくブルートフォース解析

## 実践的な手法
## CASE1
```bash
cp /etc/passwd passwd.txt
cp /etc/shadow shadow.txt
sudo unshadow passwd.txt shadow.txt > hash.txt
john --format=crypt hash.txt
```
  
### 説　明
- パスワード、シャドウファイルのコピー
  - `/etc/passwd`ファイルをと`/etc/shadow`ファイルの中身をテキストファイルにコピーする
  -　パスワード、シャドウファイルの結合
  - `unshadow`コマンドは、パスワードファイルとシャドウファイルを結合し、一つのパスワードファイルを生成する
  - 因みに`sudo unshadow /etc/passwd /etc/shadow > hash.txt`のように、テキストファイルにコピーしなくても`unshadow`自体は使うことはできる
- 解　析
  - オプションに`--format=crypt`を指定することで、解析対象ファイルがLinuxのパスワードファイルであることを明示している
  
## CASE2
```bash
meterpreter > hashdump
kali > vim hash.txt
kali > --format=NT --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```
  
### 説　明
- パスワードのハッシュを取得する
  - Metasploitでターゲットに侵入でき、meterpriterシェルを使えるとき、hashdumpコマンドでパスワードハッシュを取得できる
- ハッシュファイルを作る
  - なんでもいいが、テキストエディタでハッシュファイルを作成。今回はアナログにコピ張りで作成
- 解　析
  - `--format=NTはWindows`のパスワードファイル形式のことっぽい
  - `Wordlist`に`/rockyou`を使う  
  
![image](https://github.com/y-uoh/OSCP/assets/169203901/144b7ea4-3c62-4a4b-9d9a-f53dd2b3ca3b)  
-> `meterpriter`の`hashdump`コマンドでパスワードハッシュを取得  
  
![image](https://github.com/y-uoh/OSCP/assets/169203901/df17a63b-0359-4a5a-a933-efc182013051)  
![image](https://github.com/y-uoh/OSCP/assets/169203901/012bdc71-4e82-45a7-9691-621ea34b19b8)  
-> 今回は`vim`を使いコピ張りで`hash.txt`として保存  
  
![image](https://github.com/y-uoh/OSCP/assets/169203901/64d6ef26-2ee2-491b-aa98-bd6b7eccb7ba)
-> 解析と結果  
  
  
## 参　考
### 解析結果の参照
```bash
cat .john/john.pot
```
  
### 説　明
- 解析結果は`~/.john/john.pot`に保存されるのでそれを見る
- `~/.john/john.pot`に結果が保存されている状態でもう一度パスワード解析をしても既に解析済みといったエラーがでるので、もう一回解析したいときは`~/.john/john.pot`を消す必要がある。
  
![image](https://github.com/y-uoh/OSCP/assets/169203901/f73a66f2-f27c-4c70-ae98-5743f72bcbab)  
-> 解析結果の参照  
  
![image](https://github.com/y-uoh/OSCP/assets/169203901/b4466b5e-6420-402d-b9c7-f6cd39bdd156)  
-> 解析したファイルをもう一度解析しようとするとエラーが出る  
>デフォルトの入力エンコーディングを使用する： UTF-8  
>異なるソルトを持たない2つのパスワードハッシュをロード（NT [MD4 128/128 AVX 4x3）  
>クラック可能なパスワードハッシュが残っていない（FAQ参照)  
  
![image](https://github.com/y-uoh/OSCP/assets/169203901/518abf33-43d8-4d1f-ab13-9c57d5b7490c)  
-> もう一度解析を実行するには、`~/.john/john.pot`を消してから再度解析を実行すると上手くいく  
  
  








