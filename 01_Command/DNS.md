## DNS関係
#### 指定したドメインの名前解決
```bash
nslookup <FQDN>
```
#### PTRレコード（逆引き）を取得
```bash
nslookup -type=PTR <ipaddress>
```
`nslookup -type=PTR 123.456.789.123`

#### MXレコード（メールサーバのホスト名）を取得
```bash
nslookup -type=MX <FQDN>
```
`nslookup -type=MX example.com`

#### CNAMEレコード（エイリアス）を取得
```bash
nslookup -type=CNAME <FQDN>
```
`nslookup -type=CNAME example.com

#### 問い合わせるDNSサーバの切り替え
```cmd
nslookup  <FQDN> <任意のDNSサーバのIP>
```
`nslookup  example.com 8.8.8.8`

#### ゾーン情報の取得までのプロセス
```bash
nslookup　　//対話モードで実行
set type=ns  //nsレコードの取得
<情報を取得したいDNSのドメイン名> <任意のDNSIP> //nsレコードを取得したドメインと、問い合わせDNS
server <DNSのFQDN> //DNSサーバーを一時的に設定
ls -d <情報を取得したいDNSのドメイン名> //ls -d というオプションコマンドがゾーン情報を転送する
```
```bash
nslookup
set type=ns
zonetransfer.me 8.8.8.8
server nsztm1.digi.ninja
ls -d zonetransfer.me
```
