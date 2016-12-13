# binary100

|大ジャンル|中ジャンル     |小ジャンル|
|----------|---------------|----------|
|reverse   |stripped binary|PE        |

##0. 問題文
    Anti-Debugging
    Reverse it.
    bin
    may some AV will alert,but no problem.

##1. ファイルのダウンロード
**bin**というバイナリが置いてあるので、ダウンロードする。

##2. ファイルの概要を確認
とりあえずStirlingで覗いてみると、PE(Windows EXE)ということがわかる。    
![](https://gist.githubusercontent.com/boropon/368a26f1b6aec07633116555a517fdd8/raw/098d38e82b1fb9d41c7b7b5f70f18b3504819114/z_fileheader.png)

##3. プログラムを実行
以下のプロンプトが表示される。    
![](https://gist.githubusercontent.com/boropon/368a26f1b6aec07633116555a517fdd8/raw/098d38e82b1fb9d41c7b7b5f70f18b3504819114/z_execute.png)

##4. プログラム解析
IDAを使って、Input passwordを出力する箇所を特定する。  
文字列一覧（linuxのstringコマンドの結果みたいなの）を表示しているStringsというタブがあるので、  
そこから上記文字列を探して参照箇所へジャンプする。    
![](https://gist.githubusercontent.com/boropon/368a26f1b6aec07633116555a517fdd8/raw/098d38e82b1fb9d41c7b7b5f70f18b3504819114/z_IDA1.png)

"Your password is correct"と表示されるルートの前に"I have a pen."という文字列との比較処理があるので、  
おそらくパスワードは"I have a pen."だと推測できる。    
![](https://gist.githubusercontent.com/boropon/368a26f1b6aec07633116555a517fdd8/raw/098d38e82b1fb9d41c7b7b5f70f18b3504819114/z_IDA2.png)

パスワードが正解でも、色々な条件で不正解ルートへと弾かれてしまう。    
![](https://gist.githubusercontent.com/boropon/368a26f1b6aec07633116555a517fdd8/raw/098d38e82b1fb9d41c7b7b5f70f18b3504819114/z_IDA3.png)

条件を一通り見ていくと、それまでの処理とは趣の違う処理が見える。多分これがFLAG生成処理だと推測できる。    
![](https://gist.githubusercontent.com/boropon/368a26f1b6aec07633116555a517fdd8/raw/098d38e82b1fb9d41c7b7b5f70f18b3504819114/z_IDA4.png)

##5. デバッガ実行
OllyDbgを使って、パスワード正解直後にブレーク挿入、FLAG生成処理まで一気にジャンプする。  
※本当はIDAで実行したかったが、実行失敗したので断念…。    
![](https://gist.githubusercontent.com/boropon/368a26f1b6aec07633116555a517fdd8/raw/098d38e82b1fb9d41c7b7b5f70f18b3504819114/z_IDA5.png)
    
![](https://gist.githubusercontent.com/boropon/368a26f1b6aec07633116555a517fdd8/raw/098d38e82b1fb9d41c7b7b5f70f18b3504819114/z_IDA6.png)

続きを実行するとFLAGを表示するポップアップが現れる。    
![](https://gist.githubusercontent.com/boropon/368a26f1b6aec07633116555a517fdd8/raw/098d38e82b1fb9d41c7b7b5f70f18b3504819114/z_check.png)