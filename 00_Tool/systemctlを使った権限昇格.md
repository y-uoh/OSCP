# 前　提
- systemctlにSUIDが設定されていること
  - つまるところ、systemctlはroot権限で実行される状態にあること

# 考え方
systemctlコマンドにSUIDが設定されておりそれがroot権限で実行される状態であれば、新たなサービス（デーモン）を作成しそれをsystemctlで起動するさせることでroot権限でそのサービスが実行できるだろう考え方。  
新たなサービス（デーモン）で実行するものが、攻撃者側の任意のものであればroot権限で色々できるだろうという思考  
  
# 今回の目的
今回は`/root/root.txt`というテキストファイルにルートユーザのフラグが隠されているため、それを取得するために、`cat /root/root.txt > /tmp/output`を実行するサービスを作成し、systemctlでサービスを起動することで、`root.txt`を`/tmp`に出力して`root.txt`中身を見ることを目指す。
# SUID
[SUIDとは -Qiita](https://qiita.com/miyuki_samitani/items/129fc0680f973ab12ce9)