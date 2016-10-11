# 問題文

[Reverse]
Can your brain be a Python VM? (Please use Python 2.7)

# 概要

pythonのmarshal.dumpされたオブジェクトが渡されるので、それを実行して中身を解読する問題

# writeup

ソースを実行するとパスワードを聞かれる

```
$ python c.py  
password: aaa
Wrong password... Please try again. Do not brute force. =)
```

disを使って逆アセしてみるとmainというオブジェクトをストアしている

```
>>> import dis
>>> dis.dis(code_obj)
 1           0 LOAD_CONST               1 (<code object main at 0x1051dedb0, file "<string>", line 1>)
             3 MAKE_FUNCTION            0
             6 STORE_NAME               0 (main)
 4           9 LOAD_NAME                1 (__name__)
            12 LOAD_CONST               2 ('__main__')
            15 COMPARE_OP               2 (==)
            18 POP_JUMP_IF_FALSE       31
 5          21 LOAD_NAME                0 (main)
 ```

なので、一度問題のソースを実行してメモリに載せたうえで、mainオブジェクトの中身を逆アセしてやるとコードが見える
```
>>> exec(marshal.load(....))
>>> dis.dis(main)
 1           0 LOAD_GLOBAL              0 (chr)
              3 LOAD_CONST               1 (108)
              6 CALL_FUNCTION            1
              9 LOAD_GLOBAL              0 (chr)
             12 LOAD_CONST               1 (108)
             15 CALL_FUNCTION            1
             18 LOAD_GLOBAL              0 (chr)
             21 LOAD_CONST               2 (97)
             24 CALL_FUNCTION            1
             27 LOAD_GLOBAL              0 (chr)
             30 LOAD_CONST               3 (67)
             33 CALL_FUNCTION            1
             36 ROT_TWO             
             37 BINARY_ADD          
             38 ROT_TWO             
             39 BINARY_ADD          
             40 ROT_TWO             
             41 BINARY_ADD          
（以下略）
```

mainの中身は文字列を書き出したり入れ替えたりするコードになっている。

大体、以下のような処理をしている。

* 1-737 パスワードの文字列を作成
* 759 以降：パスワード入力を促す
* 770 以降：フラグ表示

が、詳しく追う前に、ひとまず、以下のようなスクリプトで、文字部分だけを抜き出す。

```rb
File.open('dis_dis_main.txt', 'r'){|file|
 file.each{|line|
   line =~ /\((\d+)\)/
   if $1
     print $1.to_i.chr
   end
 }
}
```

すると、以下ようになる。

```
llaC em yP aht notriv lauhcamni !eac Ini npreterP tohty ntybdocese!!!ctihN{noy woc uoc naipmoa eldnur yP nnohttyb doceni euoy rb ria}!napwssro :dorWp gnssadrow...elP  esa yrtaga .ni oD tonurbf etecro)= .
```

そのままでは読めないが、なんとなく、ctihN{noyの辺り、『hitcon{』のように見えたで、それ以降が文章なるように眺めていると、何となく法則がありそう。

* 4,4,4,3,2に区切ってそれぞれの区切りを逆から並べて、少し調整をすると

```
hitcon{Now you can compile and run Python bytecode in your brain!}
```

ちなみにパスワードは、フラグ回答後、mainのdis結果を眺めてたところ、『444234442344423423423』の順番で区切って逆にすれば良いという事が分かったで、以下。

```
Call me a Python virtual machine! I can interpret Python bytecodes!!!
```
