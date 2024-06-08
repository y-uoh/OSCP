# NSEスクリプト探索
### スクリプトが格納されているディレクトリ
`/usr/share/nmap/scripts`

### 探索要領一例
`grep -rl <探索したい文字列> /usr/share/nmap/scripts`
> `grep`で`/usr/share/nmap/scripts`ディレクトリ内にある探索文字列が含まれるファイルを抽出する。  
rオプション -> ディレクトリを指定した場合は、そのサブディレクトリ内も検索する  
lオプション -> 一致するものを含むファイル名のみ抽出  
  
### example
`grep -rl struts /usr/share/nmap/scripts`
  
> `grep`で`/usr/share/nmap/scripts`ディレクトリ内にあるファイルの文字列`struts`が含まれる行を抽出  
  
![image](uploads/c9391fd1cdef3cee8e5115430ed18e0a/image.png)

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
  
