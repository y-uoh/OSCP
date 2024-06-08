## 概　要
- Webサイトに対して、ブルートフォースのコンテンツスキャンを実行する。  
- スキャンされるのは指定したディレクトリのみ
  - そのた実践的な使い方としては、`gobuster`を実行してディレクトリを検出後、検出したディレクトリを指定して更に掘り下げて探索していく  　
  
## コマンド
```bash
gobuster dir --url http://<IPアドレス>:<port(任意)> --wordlist <wordlist_file>
```
- `dir`
  - ディレクトリを探索するおまじないみたいなもの
- `--url` `-u`
  - URLを指定
  - ポート番号は任意
- `--wordlist` `-w`
  - wordlistファイルを指定
  - `/usr/share/wordlists/`の中にいっぱいある。以下、頻出Wordlist
    - `/usr/share/wordlists/dirb/common.txt`
    - `/usr/share/wordlists/dirbuster/directory-list-2.3-mini.txt`
- `-x`
  - ファイル名を指定したり、拡張子を指定したりできる

## 実行例
```bash
gobuster dir -u http://xxx.xxx.xxx.xxx/wordlist -w /usr/share/wordlist/dirbuster/directory-list-2.3-mini.txt -x .php
```
  
## 参　考
[Content Discovery Webアプリケーションに対する偵察行為 - Qiita](https://qiita.com/Brutus/items/6b89c9518ea90695232f)  
[TryHackMeメモ - zenn](https://zenn.dev/oreha_gao/scraps/86797d33828650)  
  
  