# reversing100

|大ジャンル|中ジャンル     |小ジャンル|
|----------|---------------|----------|
|reverse   |stripped binary|PE(broken)|

##1. ファイルのダウンロード
**dataloss**というバイナリが置いてあるので、ダウンロードする。

##2. ファイルの概要を確認
fileコマンド実行。識別不能。<br>
問題文にも「データは一部破損している」みたいな記述があったので、まぁ想定の範囲内。
![](https://gist.githubusercontent.com/boropon/1ac960a9d68813d400c0e83483ddb196/raw/6e3b8108cbaefb3ecb23f3ae907e1bdc76c41fef/z_file_dataloss.png)

バイナリエディタでチラ見。よくわからんけど、なんとなくPE(Windows EXE)っぽい。
![](https://gist.githubusercontent.com/boropon/1ac960a9d68813d400c0e83483ddb196/raw/6e3b8108cbaefb3ecb23f3ae907e1bdc76c41fef/z_Stiring_PE.png)

##3. 実行はできなさそうなので、とりあえず逆アセンブル
今回はELFではなくPEなので、IDAを使用。非商用利用に限りfree版が使える。

##4. せっかくIDAを使ったので、関数コールツリーを出力
結構大きいツリーが表示されて、真面目にアセンブラを読むのは辛そうだとわかる。
![](https://gist.githubusercontent.com/boropon/1ac960a9d68813d400c0e83483ddb196/raw/6e3b8108cbaefb3ecb23f3ae907e1bdc76c41fef/z_IDA_functiontree.png)

##5. FLAGの処理っぽいところを目grep
100点問題なので、どうせわかりやすい感じでFLAGの処理があるだろうと**目grep**実行。<br>
直後にそれっぽいところを見つける。<br>
ASCIIコードで+9していくとTM...という文字列になる。<br>
![](https://gist.githubusercontent.com/boropon/1ac960a9d68813d400c0e83483ddb196/raw/6e3b8108cbaefb3ecb23f3ae907e1bdc76c41fef/z_IDA_asem.png)

##6. 回答出力スクリプトを作成してFLAGっぽい文字列をゲット
![](https://gist.githubusercontent.com/boropon/1ac960a9d68813d400c0e83483ddb196/raw/6e3b8108cbaefb3ecb23f3ae907e1bdc76c41fef/z_makeflagrb.png)
![](https://gist.githubusercontent.com/boropon/1ac960a9d68813d400c0e83483ddb196/raw/6e3b8108cbaefb3ecb23f3ae907e1bdc76c41fef/z_result.png)

FLAG文字列が少しおかしいけれど、軽く文字のインサート、ソートをしたらFLAG完成。
~~~
TMCTF{datalosswontstopus}
~~~