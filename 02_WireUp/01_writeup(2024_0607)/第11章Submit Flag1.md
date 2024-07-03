## user.txtを発見し、ファイルの中身を解答してください。
  
さすがにこれは秒だったので、感想もなく、以下の手順で実施。  
  
```bash
find / -type f -name user.txt 2>/dev/null
/home/nobunaga/user.txt

```
  
```bash
cat /home/nobunaga/user.txt
$9!injecti0n!!!

```
  
### コマンド
```bash
find / -type f -name user.txt 2>/dev/null
cat /home/nobunaga/user.txt

```
  
### フラグ
`$9!injecti0n!!!`  
