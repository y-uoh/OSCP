## Enum4Linuxとは
- SMB共有を列挙するツール
  
## 1.構　文
### 1-1.基　本
```bash
enum4linux <オプション> <ipアドレス>
```
  
### 1-2.オプション
#### -U 
- ユーザーリストの取得
  
### -M
- マシンリストの取得
  
### -N
- ネームリストダンプの取得 
- `-U`および`-M`とは異なる
  
### -S
- シェアリストの取得
  
### -P
- パスワードポリシー情報の取得
  
### -G
- グループとメンバーリストｍｐ取得
  
### -a
- 上記のすべて（完全な基本列挙）
  
### 1.3.実行例
```bash
┌──(kali㉿kali)-[~/Desktop/THM_SMB]
└─$ enum4linux 10.10.3.153
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Sun Jun  9 13:49:13 2024

 =========================================( Target Information )=========================================

Target ........... 10.10.3.153
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ============================( Enumerating Workgroup/Domain on 10.10.3.153 )============================


[+] Got domain/workgroup name: WORKGROUP


 ================================( Nbtstat Information for 10.10.3.153 )================================

Looking up status of 10.10.3.153
	POLOSMB         <00> -         B <ACTIVE>  Workstation Service
	POLOSMB         <03> -         B <ACTIVE>  Messenger Service
	POLOSMB         <20> -         B <ACTIVE>  File Server Service
	..__MSBROWSE__. <01> - <GROUP> B <ACTIVE>  Master Browser
	WORKGROUP       <00> - <GROUP> B <ACTIVE>  Domain/Workgroup Name
	WORKGROUP       <1d> -         B <ACTIVE>  Master Browser
	WORKGROUP       <1e> - <GROUP> B <ACTIVE>  Browser Service Elections

	MAC Address = 00-00-00-00-00-00

 ====================================( Session Check on 10.10.3.153 )====================================


[+] Server 10.10.3.153 allows sessions using username '', password ''


 =================================( Getting domain SID for 10.10.3.153 )=================================

Domain Name: WORKGROUP
Domain Sid: (NULL SID)

[+] Can't determine if host is part of domain or part of a workgroup


 ===================================( OS information on 10.10.3.153 )===================================


[E] Can't get OS info with smbclient


[+] Got OS info for 10.10.3.153 from srvinfo: 
	POLOSMB        Wk Sv PrQ Unx NT SNT polosmb server (Samba, Ubuntu)
	platform_id     :	500
	os version      :	6.1
	server type     :	0x809a03


 ========================================( Users on 10.10.3.153 )========================================

Use of uninitialized value $users in print at ./enum4linux.pl line 972.
Use of uninitialized value $users in pattern match (m//) at ./enum4linux.pl line 975.

Use of uninitialized value $users in print at ./enum4linux.pl line 986.
Use of uninitialized value $users in pattern match (m//) at ./enum4linux.pl line 988.

 ==================================( Share Enumeration on 10.10.3.153 )==================================


	Sharename       Type      Comment
	---------       ----      -------
	netlogon        Disk      Network Logon Service
	profiles        Disk      Users profiles
	print$          Disk      Printer Drivers
	IPC$            IPC       IPC Service (polosmb server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

	Server               Comment
	---------            -------

	Workgroup            Master
	---------            -------
	WORKGROUP            POLOSMB

[+] Attempting to map shares on 10.10.3.153


[E] Can't understand response:

tree connect failed: NT_STATUS_BAD_NETWORK_NAME
//10.10.3.153/netlogon	Mapping: N/A Listing: N/A Writing: N/A
//10.10.3.153/profiles	Mapping: OK Listing: OK Writing: N/A
//10.10.3.153/print$	Mapping: DENIED Listing: N/A Writing: N/A

[E] Can't understand response:

NT_STATUS_OBJECT_NAME_NOT_FOUND listing \*
//10.10.3.153/IPC$	Mapping: N/A Listing: N/A Writing: N/A

 ============================( Password Policy Information for 10.10.3.153 )============================



[+] Attaching to 10.10.3.153 using a NULL share

[+] Trying protocol 139/SMB...

[+] Found domain(s):

	[+] POLOSMB
	[+] Builtin

[+] Password Info for Domain: POLOSMB

	[+] Minimum password length: 5
	[+] Password history length: None
	[+] Maximum password age: 37 days 6 hours 21 minutes 
	[+] Password Complexity Flags: 000000

		[+] Domain Refuse Password Change: 0
		[+] Domain Password Store Cleartext: 0
		[+] Domain Password Lockout Admins: 0
		[+] Domain Password No Clear Change: 0
		[+] Domain Password No Anon Change: 0
		[+] Domain Password Complex: 0

	[+] Minimum password age: None
	[+] Reset Account Lockout Counter: 30 minutes 
	[+] Locked Account Duration: 30 minutes 
	[+] Account Lockout Threshold: None
	[+] Forced Log off Time: 37 days 6 hours 21 minutes 



[+] Retieved partial password policy with rpcclient:


Password Complexity: Disabled
Minimum Password Length: 5


 =======================================( Groups on 10.10.3.153 )=======================================


[+] Getting builtin groups:


[+]  Getting builtin group memberships:


[+]  Getting local groups:


[+]  Getting local group memberships:


[+]  Getting domain groups:


[+]  Getting domain group memberships:


 ===================( Users on 10.10.3.153 via RID cycling (RIDS: 500-550,1000-1050) )===================


[I] Found new SID: 
S-1-22-1

[I] Found new SID: 
S-1-5-32

[I] Found new SID: 
S-1-5-32

[I] Found new SID: 
S-1-5-32

[I] Found new SID: 
S-1-5-32

[+] Enumerating users using SID S-1-22-1 and logon username '', password ''

S-1-22-1-1000 Unix User\cactus (Local User)

[+] Enumerating users using SID S-1-5-32 and logon username '', password ''

S-1-5-32-544 BUILTIN\Administrators (Local Group)
S-1-5-32-545 BUILTIN\Users (Local Group)
S-1-5-32-546 BUILTIN\Guests (Local Group)
S-1-5-32-547 BUILTIN\Power Users (Local Group)
S-1-5-32-548 BUILTIN\Account Operators (Local Group)
S-1-5-32-549 BUILTIN\Server Operators (Local Group)
S-1-5-32-550 BUILTIN\Print Operators (Local Group)

[+] Enumerating users using SID S-1-5-21-434125608-3964652802-3194254534 and logon username '', password ''

S-1-5-21-434125608-3964652802-3194254534-501 POLOSMB\nobody (Local User)
S-1-5-21-434125608-3964652802-3194254534-513 POLOSMB\None (Domain Group)

 ================================( Getting printer info for 10.10.3.153 )================================

No printers returned.


enum4linux complete on Sun Jun  9 14:06:30 2024
```