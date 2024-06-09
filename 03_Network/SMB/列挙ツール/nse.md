## NSEでのSMB共有情報列挙テクニック
### 1.ユーザ一覧の列挙
#### `smb-enum-users.nse`というスクリプトを使う
```bash
nmap -p <ポート番号> --script=smb-enum-users.nse <ターゲットのIP or FQDN>
```
    
#### example
```bash
kali@kali# nmap -p 139 --script=smb-enum-users.nse 10.10.63.26

Starting Nmap 7.60 ( https://nmap.org ) at 2022-07-10 07:56 BST
Nmap scan report for ip-10-10-63-26.eu-west-1.compute.internal (10.10.63.26)
Host is up (0.00024s latency).

PORT    STATE SERVICE
139/tcp open  netbios-ssn
MAC Address: 02:33:E9:25:96:D3 (Unknown)

Host script results:
| smb-enum-users: 
|   SKYNET\milesdyson (RID: 1000)
|     Full name:   
|     Description: 
|_    Flags:       Normal user account

Nmap done: 1 IP address (1 host up) scanned in 0.92 seconds
```
  
### 2.共有情報の列挙
#### `smb-enum-shares.nse`というスクリプトを使う
```bash
nmap -p <ポート番号> --script=smb-enum-shares.nse <ターゲットのIP or FQDN>
```
  
#### example
```bash
kali@kali# nmap -p 139 --script=smb-enum-shares.nse 10.10.63.26

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
  
### 3.よく使う手法
```bash
nmap -p 139,445 --script=smb-enum-users.nse,smb-enum-shares.nse 10.10.63.26
```
- ポートを`139`と`445`に同時にスキャンさせる
- スクリプトを`smb-enum-users.nse`と`smb-enum-shares.nse`を同時にスキャンさせる  
