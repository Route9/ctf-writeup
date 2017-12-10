# 問題文

```
-----  RunMe.py
import sys
sys.setrecursionlimit(99999)
def f(n):
    return n if n < 2 else f(n-2) + f(n-1)
print "SECCON{" + str(f(11011))[:32] + "}"
-----
```

# フラグ

```
SECCON{65076140832331717667772761541872}
```

# Write Up

フィボナッチ数列の11011番目の先頭32桁を計算するようですがそのまま実行するといつ終わるかわかりません。

```
$ python2 RunMe.py
(終わらない)
```

そのため、一度計算したものを保存しておくようにして無駄を省きます。

```
$ cat RunMe.fast.py
import sys

sys.setrecursionlimit(99999)

def f(n, table):
    if n < 2:
        return n
    else:
        if table.has_key(n-2):
            r1 = table[n-2]
        else:
            r1 = f(n-2, table)
            table[n-2] = r1

        if table.has_key(n-1):
            r2 = table[n-1]
        else:
            r2 = f(n-1, table)
            table[n-1] = r2

        return r1 + r2

print "SECCON{" + str(f(11011, {}))[:32] + "}"

$ python2 RunMe.fast.py
SECCON{65076140832331717667772761541872}
```
