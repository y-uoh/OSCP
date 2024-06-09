## 1.NSEスクリプト探索
NSEスクリプトを探索する方法は、本体が格納されるいる`ディレクトリを探索`と、その情報が格納されている`データベースを探索`する方法の２種類がある
### スクリプトが格納されているディレクトリ
`/usr/share/nmap/scripts`
### スクリプト情報のデータベース
`/usr/share/nmap/scripts/script.db`

## 2.探索要領一例
スクリプトを探索する方法は色々あるので、具体例をいくつかあげる。  

### 2-1.grepでデータベースを探索
`grep`で抽出して探索する方法
#### format
```bash
┌──(kali㉿kali)-[~]
└─$ grep "<grepしたい文字列>" /usr/share/nmap/scripts/script.db

```

#### example
`ftp`のスクリプトを探索  
```bash
┌──(kali㉿kali)-[~]
└─$ grep "ftp" /usr/share/nmap/scripts/script.db 
Entry { filename = "ftp-anon.nse", categories = { "auth", "default", "safe", } }
Entry { filename = "ftp-bounce.nse", categories = { "default", "safe", } }
Entry { filename = "ftp-brute.nse", categories = { "brute", "intrusive", } }
Entry { filename = "ftp-libopie.nse", categories = { "intrusive", "vuln", } }
Entry { filename = "ftp-proftpd-backdoor.nse", categories = { "exploit", "intrusive", "malware", "vuln", } }
Entry { filename = "ftp-syst.nse", categories = { "default", "discovery", "safe", } }
Entry { filename = "ftp-vsftpd-backdoor.nse", categories = { "exploit", "intrusive", "malware", "vuln", } }
Entry { filename = "ftp-vuln-cve2010-4221.nse", categories = { "intrusive", "vuln", } }
Entry { filename = "tftp-enum.nse", categories = { "discovery", "intrusive", } }
Entry { filename = "tftp-version.nse", categories = { "default", "safe", "version", } }

```

### 2-2.ls -lでディレクトリを探索
`ls -l`と正規表現を使ってディレクトリの中を出力して探索
#### format
```bash
┌──(kali㉿kali)-[~]
└─$ ls -l /usr/share/nmap/scripts/*<探索したい文字列>

```
  
#### example
`ftp`のスクリプトを探索。検索文字列に`*`を付けることで正規表現で検索できる。  
```bash
┌──(kali㉿kali)-[~]
└─$ ls -l /usr/share/nmap/scripts/*ftp*
-rw-r--r-- 1 root root  4530 11月  2  2023 /usr/share/nmap/scripts/ftp-anon.nse
-rw-r--r-- 1 root root  3253 11月  2  2023 /usr/share/nmap/scripts/ftp-bounce.nse
-rw-r--r-- 1 root root  3108 11月  2  2023 /usr/share/nmap/scripts/ftp-brute.nse
-rw-r--r-- 1 root root  3272 11月  2  2023 /usr/share/nmap/scripts/ftp-libopie.nse
-rw-r--r-- 1 root root  3290 11月  2  2023 /usr/share/nmap/scripts/ftp-proftpd-backdoor.nse
-rw-r--r-- 1 root root  3768 11月  2  2023 /usr/share/nmap/scripts/ftp-syst.nse
-rw-r--r-- 1 root root  6021 11月  2  2023 /usr/share/nmap/scripts/ftp-vsftpd-backdoor.nse
-rw-r--r-- 1 root root  5923 11月  2  2023 /usr/share/nmap/scripts/ftp-vuln-cve2010-4221.nse
-rw-r--r-- 1 root root  5736 11月  2  2023 /usr/share/nmap/scripts/tftp-enum.nse
-rw-r--r-- 1 root root 10034 11月  2  2023 /usr/share/nmap/scripts/tftp-version.nse

```
### 2-3.grepでディレクトリを探索
`grep`で/usr/share/nmap/scripts`ディレクトリ内にある探索文字列が含まれるファイルを抽出する。  
`-r`:ディレクトリを指定した場合は、そのサブディレクトリ内も検索する  
`-l`:一致するものを含むファイル名のみ抽出  
  
#### format
```bash
grep -rl <探索したい文字列> /usr/share/nmap/scripts

```
  
#### example
`ftp`のスクリプトを検索  
個人的には一番見やすい  
```bash
┌──(kali㉿kali)-[~]
└─$ grep -rl ftp /usr/share/nmap/scripts
/usr/share/nmap/scripts/tftp-version.nse
/usr/share/nmap/scripts/firewall-bypass.nse
/usr/share/nmap/scripts/reverse-index.nse
/usr/share/nmap/scripts/http-barracuda-dir-traversal.nse
/usr/share/nmap/scripts/http-vuln-cve2009-3960.nse
/usr/share/nmap/scripts/tftp-enum.nse
/usr/share/nmap/scripts/http-passwd.nse
/usr/share/nmap/scripts/ftp-bounce.nse
/usr/share/nmap/scripts/snmp-ios-config.nse
/usr/share/nmap/scripts/creds-summary.nse
/usr/share/nmap/scripts/ftp-proftpd-backdoor.nse
/usr/share/nmap/scripts/banner.nse
/usr/share/nmap/scripts/ftp-syst.nse
/usr/share/nmap/scripts/ftp-vuln-cve2010-4221.nse
/usr/share/nmap/scripts/ftp-vsftpd-backdoor.nse
/usr/share/nmap/scripts/ftp-libopie.nse
/usr/share/nmap/scripts/ftp-anon.nse
/usr/share/nmap/scripts/tls-alpn.nse
/usr/share/nmap/scripts/auth-owners.nse
/usr/share/nmap/scripts/ftp-brute.nse
/usr/share/nmap/scripts/dns-zone-transfer.nse
/usr/share/nmap/scripts/script.db

```

# スクリプトの実行
### オプションなしのとき
`sudo nmap --script <実行したいスクリプトファイル> <ターゲットIP> <ターゲットPort>`
  
> sudoを付けるか付けないかは環境による。  
<ターゲットPort>は任意。明示しないと全ポートをスキャンするので、時間かかる  
  
### example
`sudo nmap --script http-vuln-cve2017-5638 192.168.x.x -p80`  
> nseスクリプト`http-vuln-cve2017-5638`を`192.168.x.x`の`ポート80`に対しスキャン  
  
![image](uploads/a80ecd2756ec8950c720e2ab38638291/image.png)  

### オプションありのとき
#### スクリプトの公式リファレンスを探す
> [スクリプトの公式リファレンス](https://nmap.org/nsedoc/scripts/)でnseスクリプトの一覧を確認できるので、その中から使用したいスクリプトを探す  
  
![image](uploads/438fd687bebe600dc569291e0ca67e07/image.png)   

#### 使用できるオプションを確認する
> スクリプトの詳細を開き、使用できるオプションを確認する。  
`Script Arguments`の欄に記載されているのが使用できるオプションとオプションの記述の仕方  
スクリプト引数ともいう  

![image](uploads/dc4570a1df94b9d33d30ae47403718cc/image.png)  
  
#### 実　行
`sudo nmap --script <スクリプト名> --script-args <オプション> <ターゲットIP> <ターゲットPort>`  
  
> オプションを使うときは、オプション名の前に`--script-args`を付けることが重要  オプションの構文はスクリプトの詳細で確認とおりに記載する  
  
#### example1
`sudo namp --script http-vuln-cve2017-5638 --script-args http-vuln-cve2017-5638.path=/struts2-showcase/showcase.action 192.168.x.x -p80`  
  
> `--script-args`の後ろがオプションになるので、この例だと`path`というオプションをつけて、スキャンターゲットの的を絞っている。  
  
#### example2
`sudo namp --script http-vuln-cve2017-5638 --script-args path=/struts2-showcase/showcase.action 192.168.x.x -p80` 

> 注目したいのが`--script-args`の後ろの書き方である。  スクリプト名を完全に入力しなくてもオプションだけで設定できるものもある。（スクリプトによりそうなので注意）  
  
