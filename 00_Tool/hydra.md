# hydra
## 概　要
> 能動的パスワードクラックを行うツール
> SSHやFTPなどのサービスや、Webアプリのログインフォームに対して攻撃が可能
> `wordlist`の使用が可能  
  
## 書　式
```bash
hydra -l <ID> -p <パスワード> <IPアドレス> <プロトコル>
hydra -l <ユーザリストファイル> -P <パスワードリスト> http:<ターゲットのパス>
```
### 代表的なオプション
- `-l`
  - ユーザ名/ID 
- `-L`
  - ユーザリストファイル 
- `-p`
  - パスワード
- `-P`
  - パスワードリストファイル   

  ## 参　考
  [kali linuxでhydraを使ってパスワードクラックをしてみた - Qiita](https://qiita.com/miya_zato/items/0c32dc71208460515e34)  
  [ハッカーはhydraでログインクラックを確認する(Kali Linux) - AIを武器にホワイトハッカーになる](https://whitemarkn.com/learning-ethical-hacker/hydra/)  
  
