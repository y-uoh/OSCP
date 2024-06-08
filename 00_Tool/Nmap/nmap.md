# ポート状態一覧
### PORT
- ポート番号/プロトコルの順で表示される
### STATE
- ポートの開放、閉鎖等の状態を表示
### SERVICE
- そのポートで起動しているサービス名を表示
  

![image](uploads/dd125b10a0f003bd67308f584c36b31a/image.png)

# STATEの見方
### open
- ポートが開放されている状態を表す。
- アプリケーションがTCP、あるいはUDPでアクティブに受け入れている状態で、多くの場合、ポートスキャンの第一の目的は、この種のポートを見つけることにある。

### closed
- ポートが閉じている状態を表す。
- Nmapのプローブパケットを受信したり応答したりと、アクセス自体は可能であるが、そこで待機しているサービスやアプリケーションは存在しない。
- この種のポートの発見は、指定したIPアドレスのホストがいるかどうか（ホスト発見）やOS検出の一環として役に立つ場合がある。
- 上述の通りclosedポートはプローブパケット自体は到達可能なので、後にその一部が開放された場合は、スキャンの対象になる可能性がある。管理者はこの種のポートもFWでブロックすることを検討する場合がある。
- 
### filtered
- この状態は、FW等のパケットフィルタによってNmapのプローブパケットがポートまで到達できないことを表し、ポートが開いているかを判別することができない状態である。
- これらのポートからは情報がほとんど得られないので、攻撃者の企てを阻むことになる。
- また、FWのファイルリング機器は、プローブパケットに応答しないでプローブパケットを破棄するだけのフィルタが多く、この場合、Nmapはプローブが破棄されたのはフィルタリングではなく、NWが混雑していることが原因とみなし、再試行を行わざるとえなくなるので、攻撃者からのスキャン速度が格段に低下する。

### unfiltered
- unfiltered状態とは、ポートにはアクセス可能だが、そのポートが開いているか閉じているかをNmapでは判別できないことを表す。
- ポートをこの状態に分類できるのは、FWルールを解読するのに使われるACKスキャンだけである。
- unfilteredポートのスキャンをその他のスキャンタイプ、例えばWindowsスキャンなどで行うと、ポートが開いているかどうかを決めるのに役立つ場合がある。

### open|filtered
- Nmapがポートをこの状態に分類するのは、対象のポートが開いているかフィルタ処理されているかを判別できない場合である。
- openポートからの応答がないタイプのスキャンには、こうしたケースが発生する。また、応答がないということは、プローブやそれが引き出した応答をパケットフィルタが破棄したことを意味する場合もある。
- そのためNmapは、対象のポートがopenなのかfilteredなのかを確実に見分けることができない。UDP、IPプロトコルなどのスキャンは、ポートをこの状態に分類する。

### closed|filtered
- この状態は、ポートが閉じているかフィルタ処理されているかを、Nmapが判断できない場合に用いられる。

# ポートスキャンテクニック
### 概　説
- Nmapリファレンスガイドに記載のスキャンテクニックは、１０個ほどある。これらの手法は併用して利用することができないが、UDPスキャン（-sU）だけは例外で、TCPス
キャンタイプのいずれか１つと組み合わせて利用可能
- ポートスキャンタイプのオプションは、覚えやすいように-sCの形式になっている。ここでCは、基本的にスキャン名のなかの目立つ文字で通常は頭文字になる。
- 例外として廃止予定のFTPバウンススキャン(-b)なども存在する。

### 留意事項
- Nmapは正確な結果を出そうと試みるが、その結果は全て、ターゲットマシンもしくはその前面にあるFWから送り返されるパケットに基づいて得られたものであるという点に留
意する必要がある。
- RFCに準拠していないホストがますます広く利用されるようになっているが、これらのホストからは当然想定される応答は返って来ない。

### 基本構文
`nmap [オプション]␣[IPアドレス]␣[Port番号]`

### ターゲットの指定・除外方法
#### ＩＰ指定
![image](uploads/1aadf5bad947dbc68982414c31b4de7a/image.png)  
  

#### ポートの指定と順番の指定
![image](uploads/34afd6038eea6e7c499ce13692fde6de/image.png)  
  
# スキャン方法の指定（SCAN TECHNIQUES）

### SYNスキャン/ステルススキャン/ハーフオープンスキャン
#### 1 オプション/サンプルコマンド
`-sS　/　>nmap␣-sS␣192.198.1.1`
#### 2 概　要
TCP CONNECT スキャンと違い、3Wayハンドシェイクで完全なセッションを結ばす、SYNパケットのみ利用してポートを確認する。ログは残らないが、実施のためroot権限が必要である
#### 3 プローブパケットの流れ
![image](uploads/f3ff5fcc5bf2ad4c672104c1ef6404b4/image.png)
　
### TCP CONNECT スキャン/TCP OPEN スキャン
#### 1 オプション/サンプルコマンド
`-sT　/　>nmap␣ｰsT␣192.198.1.1`
#### 2 概　要
スキャンを行うプロトコルとしてTCPを明示的に指定する場合に利用する。3Wayハンドシェイクを結び、オープンされているポートを確認するスキャン。信頼性が高くてroot権限がなくてもスキャンできるが、スキャン速度が遅く、ログが残る。
#### 3 プローブパケットの流れ
![image](uploads/2f2c91fb867e940c182a4884ed9b036b/image.png)

### UDPスキャン
#### 1 オプション/サンプルコマンド
`-sU　/　>nmap␣ｰsU␣192.198.1.1`
#### 2 概　要
UDPを明示的に指定する場合に利用。UDPプロトコルを利用してポートスキャンを行う。ポートがオープンされている場合、ターゲットからは応答はないが、クローズされている場合は、ICMPメッセージ（Destination Unreachable,Port Unreachable）の応答がある。但し、UDPスキャンはルータやFWからパケット損失される可能性が高いため、信頼性は低い。
#### 3 プローブパケットの流れ
![image](uploads/f0b0dad5a3ec1603cf18fed6a1190400/image.png)  


　