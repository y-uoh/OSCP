### 1.Anonymousユーザ
- FTPと同じで`パスワード認証なし`でログインできる`Anonymousユーザ`が存在する。（もちろん設定による）  

### 2.Anonymousがいるか探索手法
#### 2-1.`nse`の`smb-enum-users.nse`及び`smb-enum-shares.nse`を使う（おススメ）
example：`10.10.10.10`の`139番ポート`に対し`--script=smb-enum-shares.nse`を実行
```bash
kali@kali# nmap -p 139 --script=smb-enum-shares.nse 10.10.10.10

Starting Nmap 7.60 ( https://nmap.org ) at 2022-07-10 07:58 BST
Nmap scan report for ip-10-10-63-26.eu-west-1.compute.internal (10.10.63.26)
Host is up (0.00022s latency).

PORT    STATE SERVICE
139/tcp open  netbios-ssn
MAC Address: 02:33:E9:25:96:D3 (Unknown)

Host script results:
| smb-enum-shares: 
|   account_used: guest
|   \\10.10.63.26\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (skynet server (Samba, Ubuntu))
|     Users: 1
|     Max Users: <unlimited>
|     Path: C:\tmp
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.63.26\anonymous: 
|     Type: STYPE_DISKTREE
|     Comment: Skynet Anonymous Share
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\srv\samba
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.63.26\milesdyson: 
|     Type: STYPE_DISKTREE
|     Comment: Miles Dyson Personal Share
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\home\milesdyson\share
|     Anonymous access: <none>
|     Current user access: <none>
|   \\10.10.63.26\print$: 
|     Type: STYPE_DISKTREE
|     Comment: Printer Drivers
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\var\lib\samba\printers
|     Anonymous access: <none>
|_    Current user access: <none>

Nmap done: 1 IP address (1 host up) scanned in 1.19 seconds
```
  
### 2-2.Anonymousでログインを試してみる
- 通常のログインのように、`smbclient //<IP>/<共有フォルダ> -U Anonymous`でログインを試してみる
- 入れたら`Anonymous`がいるという`威力偵察探索`
```bash
smbclient //10.1.1.1/share -U Anonymous
```
  
### 3.Anonymousログインイメージ
- 通常のログインのように、`smbclient //<IP>/<共有フォルダ> -U Anonymous`でログインする
- 一応パスワードを聞かれるが、何も入力せずそのまま`Enter`する。
- ログインに成功すれば`Try "help" to get a list of possible commands.`が出てきて、プロンプトが`smb: \>`で戻ってくる。
```bash
┌──(kali㉿kali)-[~]
└─$ smbclient //10.10.3.153/profiles -U Anonymous              
Password for [WORKGROUP\Anonymous]:
Try "help" to get a list of possible commands.
smb: \> 
```