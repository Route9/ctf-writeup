# 概要
javascriptを読んで、パスワードを逆算していく問題

# 手順
## 1 ファイルが何かを確かめる

```sh
$ file tmctf.hta 
tmctf.hta: HTML document text
```

## 2 ファイルを開いて眺める

* "CTF"で検索すると、以下の関数がある

```js
function checkPW(pass) {
    if (pass != null && pass == "close") {
        window.close();
    };
    if (pass == null || pass.length != 24) {
        alert("Wrong password");
        return;
    };
    if (pass.substring(0, 6) != "TMCTF{" || pass.substr(pass.length - 1) != "}") {
        alert("Wrong password");
        return;
    };
    var pwbody = (" " + pass.substring(6, pass.length - 1)).split("");
    var h1 = "",
        h2 = "",
        h3 = "";
    for (var i = 0; i < pwbody.length;) {
        h1 += pwbody[nl[++i]];
        h2 += pwbody[nl[++i]];
        h3 += pwbody[nl[++i]];
    };
    if (co(m(h1.replace(/(^¥s+)|(¥s+$)/g, ""))) && ca(m(h3.replace(/(^¥s+)|(¥s+$)/g, ""))) && cq(m(h2.replace(/(^¥s+)|(¥s+$)/g, "")))) {
        alert("ok!");
        window.close();
    } else {
        alert("Wrong password");
        return;
    };
}
```

## 3 コードを読んで行く
```js
co(m(h1.replace(/(^¥s+)|(¥s+$)/g, "")))
```
* h1は文字列であり、replaceは、前後の空白を削る処理
* m()はMD5を求る処理
* h3, h2についても同様の事をしている

### co(), ca(), cq()
それぞれ、引数を何かと比較している
```js
function co(o) {
    return (o === ko);
}

function ca(a) {
    return (a === ka.replace(/6/g, '2').replace(/b/g, 'a').replace(/d/g, '4'));
}

function cq(q) {
    return (q === kq.replace(/1/g, '5').replace(/2/g, '8').replace(/3/g, 'b').replace(/9/g, 'c'));
}
```

コードのもっと下の方で色々と作り込む処理をしているが、面倒なので、ファイルをchromeで開き、デバッグ用のコンソールから、それぞれの値を確認する

```js
>ko
"c33367701511b4f6020ec61ded352059"
>ka.replace(/6/g, '2').replace(/b/g, 'a').replace(/d/g, '4')
"21232f297a57a5a743894a0e4a801fc3"
>kq.replace(/1/g, '5').replace(/2/g, '8').replace(/3/g, 'b').replace(/9/g, 'c')
"d8578edf8458ce06fbc5bb76a58c5ca4"
```

* やはり32個の16進数で、MD5なので、適当にwebで検索をすると、それぞれ以下である事がわかる
 * [c33367701511b4f6020ec61ded352059](http://md5cracker.org/decrypted-md5-hash/c33367701511b4f6020ec61ded352059) => 654321
 * [21232f297a57a5a743894a0e4a801fc3](http://md5cracker.org/decrypted-md5-hash/21232f297a57a5a743894a0e4a801fc3) => admin
 * [d8578edf8458ce06fbc5bb76a58c5ca4](http://md5cracker.org/decrypted-md5-hash/d8578edf8458ce06fbc5bb76a58c5ca4) => qwerty

つまり、h1は654321, h3はadmin, h2はqwerty

### h1-3を考える
h1-3は少し上の部分で、nl配列に従ってpwbodyから取得し、振り分けられている
```js
for (var i = 0; i < pwbody.length;) {
    h1 += pwbody[nl[++i]];
    h2 += pwbody[nl[++i]];
    h3 += pwbody[nl[++i]];
};
```
* ++iなので、nl配列を1番目から利用する

```
>nl
[0, 2, 1, 12, 7, 15, 5, 4, 8, 16, 17, 3, 9, 10, 14, 11, 13, 6, 0]
```
これが、"6qa5wd4em3ri2tn1y"と対応するので、nlの数値と突き合わせるとpwbodyは、
"q6r4dy5ei2na1twm3"となる

最終的な答えは、
```
TMCTF{q6r4dy5ei2na1twm3}
```
となる
