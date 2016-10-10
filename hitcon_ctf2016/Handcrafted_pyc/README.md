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
文字列を復元してもそのままでは読めないが、4,4,4,3,2に区切ってそれぞれの区切りを逆から並べると

hitcon{Now you can compile and run Python bytecode in your brain!}

ちなみにパスワードは Call me a Python virtual machine! I can interpret Python bytecodes!!!