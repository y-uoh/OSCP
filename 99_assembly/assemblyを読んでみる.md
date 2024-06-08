# ツールについて
### Compiler Explorer
[Compiler Explorer](https://godbolt.org/)は、.cファイルをコンパイルし、.sファイルに変換してくれるサイト。左ウィンドウにソースコードを記述すると右ウィンドウにアセンブリに変換される。

# 読んでみるソースコード
Compiler Explorerを使用して、以下のC言語のソースコードをコンパイルする。  
内容は、`変数aに1`を、`変数bに2`を代入して`a+bの結果を出力`するというのと、  
`hello worldを出力`するというプログラム  

```c
#include <stdio.h>
int main()
{
    int a=1;
    int b=2;
    printf("%d\n", a+b);
    printf("hello world");
    return 0;
}
```
以下をコンパイルしアセンブリコードになったのが以下  

```nasm
.LC0:
        .string "%d\n"
.LC1:
        .string "hello world"
main:
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16
        mov     DWORD PTR [rbp-4], 1
        mov     DWORD PTR [rbp-8], 2
        mov     edx, DWORD PTR [rbp-4]
        mov     eax, DWORD PTR [rbp-8]
        add     eax, edx
        mov     esi, eax
        mov     edi, OFFSET FLAT:.LC0
        mov     eax, 0
        call    printf
        mov     edi, OFFSET FLAT:.LC1
        mov     eax, 0
        call    printf
        mov     eax, 0
        leave
        ret
```

前提知識を基に、今回はこのアセンブリコードを読んでみる。  
なお、前提知識で出てこなかったものは適宜補足する。  

### Local Constantとラベル
最初に出てくるこの部分
```nasm
.LC0:
        .string "%d\n"
.LC1:
        .string "hello world"
```
この`.LC0:`や`.LC1:`は、ラベルと呼ばれ、メモリ領域に名前を付したもののことを指す。  
`LOn:`は、**Local Constant**の略のことをいう。大抵ドットで始まるラベルはコンパイラが用意した**ローカルラベル**と呼ばれるもので、**自分で定義した関数やmainだとドットがつかない。**  

#### Local Constant
文字列などの定数を表す。  

#### ラベル
メモリ上の場所を名前付けしたもので、コンパイル時にメモリのアドレスに変化する。  
`main:`はmainラベルで、int main()部分に対応。つまるところmainが持っているメモリ領域

### mainラベル以下
次に出てくるmainラベル以下の部分
```nasm
main:
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16
```

ここで、メモリのスタック領域に場所を確保しており、**Function Prologue**と呼ばれる。以下は、このFunction Prologueを一行ずつ説明する。スタックの動きについては下図を参照  
![image](uploads/cd81328609730d57ef41563314db118d/image.png)　　

#### push命令
```nasm
push rbp
```

- pushは、レジスタの値をスタックに積む。ここではrbpをスタックに積んでいる。なおこのとき、スタックの先頭であるrspも更新される。  
- 図では、(1) -> (2)

#### mov命令
```nasm
mov rbp, rsp
```
- rspをrbpに代入する操作
- 図では、(2) -> (3)  

#### sub命令
```nasm
sub rsp, 16
```

- rspから16を引いて領域を
