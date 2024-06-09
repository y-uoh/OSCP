## 1.mountとは
- mountはディスク装置をLinuxのディレクトリ内に埋め込み、使えるようにすること
- 簡単に言うと、NFSやSMBなどのリモート上の共有ディレクトリをローカルディレクトリと接続するようなイメージ
- USBなどの物理ディスクはコンピュータにセットした時点で適切な場所に自動でマウントし、使えるようにしてくれる。
  
## 2.mountコマンド
- mountコマンドは、mountに関する操作や管理を行う際に使用するコマンド
  
### 2-1.基本動作
```bash
mount
```
#### 現在のマウント状況を確認できる：  
- マウントされているデバイス情報
- デバイスタイプ
- マウントポイント（マウントされているディレクトリとそのパスのこと）
  
#### example：
```bash
┌──(kali㉿kali)-[~]
└─$ mount                                                          
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
udev on /dev type devtmpfs (rw,nosuid,relatime,size=965292k,nr_inodes=241323,mode=755,inode64)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
tmpfs on /run type tmpfs (rw,nosuid,nodev,noexec,relatime,size=201516k,mode=755,inode64)
/dev/sda1 on / type ext4 (rw,relatime,errors=remount-ro)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev,inode64)
tmpfs on /run/lock type tmpfs (rw,nosuid,nodev,noexec,relatime,size=5120k,inode64)
cgroup2 on /sys/fs/cgroup type cgroup2 (rw,nosuid,nodev,noexec,relatime,nsdelegate,memory_recursiveprot)
pstore on /sys/fs/pstore type pstore (rw,nosuid,nodev,noexec,relatime)
bpf on /sys/fs/bpf type bpf (rw,nosuid,nodev,noexec,relatime,mode=700)
systemd-1 on /proc/sys/fs/binfmt_misc type autofs (rw,relatime,fd=32,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=1640)
hugetlbfs on /dev/hugepages type hugetlbfs (rw,nosuid,nodev,relatime,pagesize=2M)
debugfs on /sys/kernel/debug type debugfs (rw,nosuid,nodev,noexec,relatime)
tracefs on /sys/kernel/tracing type tracefs (rw,nosuid,nodev,noexec,relatime)
mqueue on /dev/mqueue type mqueue (rw,nosuid,nodev,noexec,relatime)
configfs on /sys/kernel/config type configfs (rw,nosuid,nodev,noexec,relatime)
fusectl on /sys/fs/fuse/connections type fusectl (rw,nosuid,nodev,noexec,relatime)
vmware-vmblock on /run/vmblock-fuse type fuse.vmware-vmblock (rw,relatime,user_id=0,group_id=0,default_permissions,allow_other)
binfmt_misc on /proc/sys/fs/binfmt_misc type binfmt_misc (rw,nosuid,nodev,noexec,relatime)
sunrpc on /run/rpc_pipefs type rpc_pipefs (rw,relatime)
tmpfs on /run/user/1000 type tmpfs (rw,nosuid,nodev,relatime,size=201512k,nr_inodes=50378,mode=700,uid=1000,gid=1000,inode64)
gvfsd-fuse on /run/user/1000/gvfs type fuse.gvfsd-fuse (rw,nosuid,nodev,relatime,user_id=1000,group_id=1000)
10.10.202.253:/home on /tmp/mount type nfs4 (rw,relatime,vers=4.2,rsize=131072,wsize=131072,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.9.0.5,local_lock=none,addr=10.10.202.253)
```
  
## 3.オプション
### 3-1.`-a`：設定されているファイルシステムを全てマウントする
```bash
mount -a
```
Linuxに標準でマウントされるデバイスは、`/etc/fstab`の中で設定されており、ここに設定されている全てのデバイスを一括でマウントしたいときに使用する。  

  
### 3-2.`-t`：マウントするデバイスタイプを指定する
```bash
mount -t <デバイスタイプ> <デバイス名 or マウントしたディレクトリ名> <マウントポイント>
```
　デバイスタイプは数多く存在し、それぞれ最大容量や１ファイルの最小ファイルサイズ、セキュリティレベルが違う。  
そのため`デバイスタイプを指定してマウント`するのは標準的な方法であり、マウントコマンドの標準的なの構文は`-t`を使用するのが一般的  
　マウントポイントとは、マウント先のディレクトリのことでありあらかじめ`mkdir`などで作成することが多い。
  
#### 代表的は`デバイスタイプ`の一覧
- `ext4`：Linuxのファイルシステム
- `ext3`,`ext4`：少し前のLinuxのファイルシステム
- `msdos`：MS-DOSのファイルシステム
- `vfat`：vfatのファイルシステム
- `iso9660`：CD-ROMなどの光学ディスク全般
- `nfs`：ネットワークファイルシステム  
  
#### example
```bash
┌──(kali㉿kali)-[~]
└─$ mount -t nfs 10.10.202.253:home /tmp/mount -nolock
```
- `-t`でデバイスタイプに`nfs`を指定
- nfsの共有ディレクトリのマウントは、マウントしたいディレクトリの部分に`nfsサーバのIP`と`マウントしたいディレクトリ名`を`:`で区切って指定する。ここでは、`10.10.202.253`のnfsサーバーの`home`という共有ディレクトリをマウントする。
- マウントポイントに`/tmp/mount`を指定
- おまけ：`-nolock`は`ファイルロックを無効`にするオプション
  
### 3-3.`-r`：読み出し専用/read onlyでマウントする
```bash
mount -r -t <デバイスタイプ> <マウントしたいデバイス名 or マウントしたいディレクトリ名> <マウントポイント>
```
単純に`読み出し専用/read only`でマウントするオプション  
   
#### example
```bash
$ mount -r -t iso9660 /dev/cdrom /media/cd
```
- `-t`でデバイスタイプに`光学ディスク`を指定
- マウントしたいデバイス名`/dev/cdrom`を指定
- マウントポイントに`/media/cd`を指定  
  
### 3-4.`-w`：読み書き可能/read writeでマウントする
```bash
mount -w -t <デバイスタイプ> <マウントしたいデバイス名 or マウントしたいディレクトリ名> <マウントポイント>
```
単純に`読み書き可能/read write`でマウントするオプション  
   
#### example
```bash
$ mount -r -t iso9660 /dev/cdrom /media/cd
```
- `-t`でデバイスタイプに`光学ディスク`を指定
- マウントしたいデバイス名`/dev/cdrom`を指定
- マウントポイントに`/media/cd`を指定  
  
### オプションまとめ
- `-a`：設定されているファイルシステム全てをマウント
- `-t`：デバイスタイプを指定する（標準的な構文）
- `-r`：読み出し専用/read onlyでマウントする
- `-w`：読み書き可能/read writeでマウントする  
  
  