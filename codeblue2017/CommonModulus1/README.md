# 問題文

We made RSA Encryption Scheme/Tester. Can you break it?

[Common\_Modulus\_1.zip](https://static.score.ctf.codeblue.jp/attachments/Common_Modulus_1.zip-37882dbd7dd05381bbf72a11fbbdb3f23def0e4981bc9ffcd399e4c138549fc8)

## フラグ

```
CBCTF{6ac2afd2fc108894db8ab21d1e30d3f3}
```

## 解き方

まずはダウンロードしたファイルの中身を確認します。

```
$ unzip Common_Modulus_1.zip
Archive:  Common_Modulus_1.zip
   creating: Common_Modulus_1/
  inflating: Common_Modulus_1/problem.py
  inflating: Common_Modulus_1/transcript.txt
```

Common\_Modulus\_1/transcript.txtを眺めると以下のような2つの暗号文？がある事がわかります。

```
[+] RSA Self Test: (791...147L, 813647) # RSA で何かしら行った際のデバッグ情報？(左がnで右がe？
[+] ciphertext = 767...                 # 暗号文？
[+] Dec(Enc(m)) == m? : True            # 暗号文を復号したものが正しく復号できているかどうか？
[+] RSA Self Test: (791...147L, 846359)
[+] ciphertext = 393...
[+] Dec(Enc(m)) == m? : True
```

次にproblem.pyを読みます。

以下のコードより、keyモジュールの下にフラグが定義されている気がしてきます。

```python
import key

FLAG = long(key.FLAG.encode("hex"), 16)
```

しかし、[keyモジュール？](https://pypi.python.org/pypi/key/0.4)自体は
実在するものの、当然[ソースコード](https://github.com/awans/key)を見て
もフラグは定義されていないので作成した環境からパスか何かが通ったところ
に置いてある全く別の自作モジュールという気分になります。

なお、以下のようなのでFLAGは一旦は単なる大きな整数になっています。

1. `key.FLAG.encode("hex")` でフラグは何かしらのルールで16進数の文字列に変換される
2. `long(key.FLAG.encode("hex"), 16)` で1.の結果が16進の長整数？として解釈される

次に以下を見てみます。

```python
def test(n, p, q):
  e = get_random_prime(20)
  pk, sk = (n, e), (long(gmpy.invert(e, (p-1)*(q-1))), )
  print "[+] RSA Self Test: %r" % (pk, )
  c = encrypt(pk, FLAG)
  print "[+] ciphertext = %d" % c
  m = decrypt(pk, sk, c)
  print "[+] Dec(Enc(m)) == m? : %s" % (m == FLAG)
```

以下の事がわかります。

* nとeの予想は合っている
    * この書き方はRubyでいう多重代入？
* n、p、qは2回のtestで同一だがeは毎回生成している
* Pythonは2系(printに括弧がないからだけど合っているかはわからない)？

nとeはRSAでは公開鍵として公開されてもいい情報ですが、普通変えないeだけ
違うものをわざわざ使っている事が気になります。そのあたりをキーワードを
変えながら検索するとCommon Modulus Attackが効きそう？です。

[ひとさまのサイト](http://inaz2.hatenablog.com/entry/2016/01/15/011138)を参考に解読して逆を辿って文字列にしてみるとフラグが取れました。

```
$ pip install gmpy
$ cat solve.py
import sys
import gmpy
import binascii

def common_modulus_attack(c1, c2, e1, e2, n):
    gcd, s1, s2 = gmpy.gcdext(e1, e2)
    if s1 < 0:
        s1 = -s1
        c1 = gmpy.invert(c1, n)
    elif s2 < 0:
        s2 = -s2
        c2 = gmpy.invert(c2, n)
    v = pow(c1, s1, n)
    w = pow(c2, s2, n)
    m = (v * w) % n
    return m

if __name__ == '__main__':
    n  = 791311...
    e1 = 846359
    c1 = 393205...
    e2 = 813647
    c2 = 767202...

    flag_hex = hex(common_modulus_attack(c1, c2, e1, e2, n))
    print "flag(hex): %r" % flag_hex
    print "flag(ascii): " + binascii.unhexlify(flag_hex[2:])

$ python solve.py
flag(hex): '0x43424354467b36616332616664326663313038383934646238616232316431653330643366337d'
flag(ascii): CBCTF{6ac2afd2fc108894db8ab21d1e30d3f3}
```
