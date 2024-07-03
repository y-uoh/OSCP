## root権限で実行できるように設定されているファイルを探すと、任意のファイルを上書きできる実行ファイルが見つかる。その実行ファイル名を解答してください。  
  
ほんと`SUID`の恐ろしさと、`find`の使い勝手の良さったらないよね  
ここもあんま悩むことなく進めた。  
まず、`find`で`SUID`に条件絞って検索  
  
```bash
find / -perm -4000 2>/dev/null
/snap/snapd/16292/usr/lib/snapd/snap-confine
/snap/core20/1623/usr/bin/chfn
/snap/core20/1623/usr/bin/chsh
/snap/core20/1623/usr/bin/gpasswd
/snap/core20/1623/usr/bin/mount
/snap/core20/1623/usr/bin/newgrp
/snap/core20/1623/usr/bin/passwd
/snap/core20/1623/usr/bin/su
/snap/core20/1623/usr/bin/sudo
/snap/core20/1623/usr/bin/umount
/snap/core20/1623/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core20/1623/usr/lib/openssh/ssh-keysign
/snap/core18/2560/bin/mount
/snap/core18/2560/bin/ping
/snap/core18/2560/bin/su
/snap/core18/2560/bin/umount
/snap/core18/2560/usr/bin/chfn
/snap/core18/2560/usr/bin/chsh
/snap/core18/2560/usr/bin/gpasswd
/snap/core18/2560/usr/bin/newgrp
/snap/core18/2560/usr/bin/passwd
/snap/core18/2560/usr/bin/sudo
/snap/core18/2560/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core18/2560/usr/lib/openssh/ssh-keysign
/usr/bin/chfn
/usr/bin/umount
/usr/bin/gpasswd
/usr/bin/mount
/usr/bin/su
/usr/bin/fusermount3
/usr/bin/passwd
/usr/bin/cp
/usr/bin/chsh
/usr/bin/newgrp
/usr/bin/pkexec
/usr/bin/sudo
/usr/libexec/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine


```
  
そういやプロンプト正常化させてないや。  
みずらいかな？  
  
こういうとき、明らかに怪しいやつの相場は下の方って決まってるって俺が言ってた。  
とまぁ実際になんで`cp`コマンドに`SUID`ついてんねんって話しで、予想通りそのまま`submit`してクリア  
  
## コマンド
```bash
find / -perm -4000 2>/dev/null

```
  
## フラグ
`cp`  
  
  
