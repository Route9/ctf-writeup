# 問題文

[Web200]
Remote Code Execution!

# 概要
用意されたサーバーに対してコードを渡し、実行してフラグを取得する問題。

# writeup

## 前提

- サーバーはNode.jsで実行されている。
- クエリパラメータ`data`に渡したコード(length <= 12)がNode.jsサーバーのvm2モジュールによって実行される。

## 狙いどころ

### リモート実行コード内で利用可能なグローバルオブジェクト

`vm._context`の中身

```
Buffer: Proxy
GLOBAL: Object
global: Object
isVM: true
root: Object
__proto__: Object
```

これらのプロパティ + プリミティブなオブジェクトを実行コード中では利用することが可能。


### NodeのBufferオブジェクトの注意点
https://nodejs.org/api/buffer.html#buffer_what_makes_buffer_allocunsafe_and_buffer_allocunsafeslow_unsafe

> When calling Buffer.allocUnsafe() and Buffer.allocUnsafeSlow(), the segment of allocated memory is uninitialized (it is not zeroed-out). While this design makes the allocation of memory quite fast, the allocated segment of memory might contain old data that is potentially sensitive. Using a Buffer created by Buffer.allocUnsafe() without completely overwriting the memory can allow this old data to be leaked when the Buffer memory is read.

Node.jsのオフィシャルなドキュメントにはこのように書いてあり、**shared internal memory pool**が共有されることで、メモリの内容がリークしてしまうことを示唆している。

この現象が発生するのは、以下の呼び出しであり、今回のケースではメモリ内にflagが存在している状態となっていた。

- `new Buffer(size)`
- `Buffer.allocUnsafe(size)`


### メモリ内容の出力

以下のクエリパラメータにより、`Buffer`クラスのコンストラクタを呼び出す。
この呼び出しにより、flag文字列の含まれたファイルが出力される。

http://52.198.115.130:3000/?data=Buffer(9999)

