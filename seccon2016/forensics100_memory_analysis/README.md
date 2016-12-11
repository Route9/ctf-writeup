# 問題文
```
Memory Analysis
Find the website that the fake svchost is accessing.
You can get the flag if you access the website!!

memoryanalysis.zip
The challenge files are huge, please download it first. 
Hint1: http://www.volatilityfoundation.org/
Hint2: Check the hosts file
```

# writeup

コマンドの実行結果は、適当に一部だけを切り出して記載しています。

## profileを見る

```
$ ./volatility-2.5.standalone.exe -f forensic_100.raw imageinfo
Volatility Foundation Volatility Framework 2.5
INFO    : volatility.debug    : Determining profile based on KDBG search...
          Suggested Profile(s) : WinXPSP2x86, WinXPSP3x86 (Instantiated with WinXPSP2x86)
           Image date and time : 2016-12-06 05:28:47 UTC+0000
     Image local date and time : 2016-12-06 14:28:47 +0900
```

## プロセス一覧を眺めてみる

```
$ ./volatility-2.5.standalone.exe -f forensic_100.raw pstree
Volatility Foundation Volatility Framework 2.5
Name                                                  Pid   PPid   Thds   Hnds Time
-------------------------------------------------- ------ ------ ------ ------ ----
 0x823c8660:System                                      4      0     58    259 1970-01-01 00:00:00 UTC+0000
. 0x81a18020:smss.exe                                 540      4      3     19 2016-12-06 05:27:04 UTC+0000
.. 0x82173da0:winlogon.exe                            628    540     24    541 2016-12-06 05:27:07 UTC+0000
... 0x8216e670:services.exe                           672    628     15    286 2016-12-06 05:27:07 UTC+0000
.... 0x81f65da0:svchost.exe                          1776    672      2     23 2016-12-06 05:27:10 UTC+0000
..... 0x8225bda0:IEXPLORE.EXE                         380   1776     22    385 2016-12-06 05:27:19 UTC+0000
...... 0x8229f7e8:IEXPLORE.EXE                       1080    380     19    397 2016-12-06 05:27:21 UTC+0000
```

起動時刻や親子関係や名前を眺めていると、上記の部分がIEがIEを生み出しているように見えて、なんとなく怪しい気がするので、以下を覚えておく。
* pid 1080
* 2016/12/06 05:27:21

※ 正常動作時を知らないので、実は正しい可能性はある

### 余談
参考サイトによると、今回の問題では無かったが、pstree時に、svchost.exeがservices.exeと同階層にあったりするのも怪しい兆候らしい。

また以下のように「Vad Tag: VadS Protection: PAGE_EXECUTE_READWRITE」があると、プロセスがインジェクションされている兆候らしい。

```
$ ./volatility-2.5.standalone.exe -f forensic_100.raw --profile=WinXPSP2x86 malfind -p 1080
Volatility Foundation Volatility Framework 2.5
Process: IEXPLORE.EXE Pid: 1080 Address: 0x780000
Vad Tag: VadS Protection: PAGE_EXECUTE_READWRITE
Flags: CommitCharge: 2, MemCommit: 1, PrivateMemory: 1, Protection: 6

0x00780000  b0 00 eb 70 b0 01 eb 6c b0 02 eb 68 b0 03 eb 64   ...p...l...h...d
0x00780010  b0 04 eb 60 b0 05 eb 5c b0 06 eb 58 b0 07 eb 54   ...`...\...X...T
0x00780020  b0 08 eb 50 b0 09 eb 4c b0 0a eb 48 b0 0b eb 44   ...P...L...H...D
0x00780030  b0 0c eb 40 b0 0d eb 3c b0 0e eb 38 b0 0f eb 34   ...@...<...8...4

Process: IEXPLORE.EXE Pid: 1080 Address: 0x5fff0000
Vad Tag: VadS Protection: PAGE_EXECUTE_READWRITE
Flags: CommitCharge: 16, MemCommit: 1, PrivateMemory: 1, Protection: 6

0x5fff0000  64 74 72 52 00 00 00 00 20 03 ff 5f 00 00 00 00   dtrR......._....
0x5fff0010  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   ................
0x5fff0020  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   ................
0x5fff0030  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   ................
```

## 通信一覧を眺めてみる

```
$ ./volatility-2.5.standalone.exe -f forensic_100.raw connscan
Volatility Foundation Volatility Framework 2.5
Offset(P)  Local Address             Remote Address            Pid
---------- ------------------------- ------------------------- ---
0x018c3cc8 192.168.88.131:1077       180.70.134.87:80          3676
0x0196f6a0 192.168.88.131:1122       175.126.170.70:80         3676
0x0233bbe8 192.168.88.131:1034       153.127.200.178:80        1080
0x02470238 192.168.88.131:1036       172.217.27.78:443         2776
```

connscanなので、closedなものも出るが、pid 1080が存在している。
* connectionsプラグインを叩くと、通信中のものだけが見えるが、その際はpid 1080のものだけになる

試しに、153.127.200.178にアクセスすると、nginx/1.10.0をインストールしただけのようなページが表示される。

https://www.shodan.io/host/153.127.200.178 を見てみると、SAKURAを利用しているようで、何となく問題用のサーバーっぽい雰囲気がある。


## タイムラインを生成してみる

```
$ ./volatility-2.5.standalone.exe -f forensic_100.raw --profile=WinXPSP2x86 timeliner
2016-12-06 03:39:11 UTC+0000|[IEHISTORY]| IEXPLORE.EXE->http://crattack.tistory.com/entry/Data-Science-import-pandas-as-pd| PID: 1080/Cache type "URL " at 0x194be00 End: 2016-12-06 05:28:40 UTC+0000
2016-12-06 03:39:11 UTC+0000|[IEHISTORY]| IEXPLORE.EXE->http://crattack.tistory.com/entry/Data-Science-import-pandas-as-pd| PID: 1080/Cache type "LEAK" at 0x194bb00 End: 2016-12-06 05:19:13 UTC+0000
2016-12-06 03:39:11 UTC+0000|[IEHISTORY]| IEXPLORE.EXE->http://crattack.tistory.com/entry/Data-Science-import-pandas-as-pd| PID: 1080/Cache type "LEAK" at 0x194bc80 End: 2016-12-06 05:21:54 UTC+0000
2016-12-06 05:22:04 UTC+0000|[IEHISTORY]| IEXPLORE.EXE->Visited: SYSTEM@http://crattack.tistory.com/rss| PID: 1080/Cache type "URL " at 0x1965000 End: 2016-12-06 05:22:04 UTC+0000
2016-12-06 05:28:40 UTC+0000|[IEHISTORY]| IEXPLORE.EXE->Visited: SYSTEM@http://crattack.tistory.com/entry/Data-Science-import-pandas-as-pd| PID: 1080/Cache type "URL " at 0x1965100 End: 2016-12-06 05:28:40 UTC+0000
2016-12-06 05:28:40 UTC+0000|[IEHISTORY]| IEXPLORE.EXE->Visited: SYSTEM@http://crattack.tistory.com/entry/Data-Science-import-pandas-as-pd| PID: 1080/Cache type "URL " at 0x1965300 End: 2016-12-06 05:28:40 UTC+0000
2016-12-06 14:28:40 UTC+0000|[IEHISTORY]| IEXPLORE.EXE->:2016120620161207: SYSTEM@http://crattack.tistory.com/entry/Data-Science-import-pandas-as-pd| PID: 1080/Cache type "URL " at 0x2375000 End: 2016-12-06 05:28:40 UTC+0000
2016-12-06 14:15:02 UTC+0000|[IEHISTORY]| IEXPLORE.EXE->:2016120620161207: SYSTEM@:Host: crattack.tistory.com| PID: 1080/Cache type "URL " at 0x2375100 End: 2016-12-06 05:15:02 UTC+0000
2016-12-06 14:28:05 UTC+0000|[IEHISTORY]| IEXPLORE.EXE->:2016120620161207: SYSTEM@http://crattack.tistory.com/entry/Data-Science-import-pandas-as-pd| PID: 1080/Cache type "URL " at 0x2375200 End: 2016-12-06 05:28:05 UTC+0000
```
pidと時刻から、http://crattack.tistory.com/entry/Data-Science-import-pandas-as-pd にアクセスしていたと思われる。

しかし、このサイトは韓国のセキュリティに関するサイトのようで、問題とは関連がない。

## ヒントにhostsファイルを見ろとあるので、hostsファイルを抽出する

```
$ ./volatility-2.5.standalone.exe -f forensic_100.raw --profile=WinXPSP2x86 filescan | grep -i host
Volatility Foundation Volatility Framework 2.5
0x000000000201ef90      1      0 R--rw- \Device\HarddiskVolume1\WINDOWS\system32\svchost.exe
0x00000000020f0268      1      0 R--r-d \Device\HarddiskVolume1\WINDOWS\svchost.exe
0x000000000217b748      1      0 R--rw- \Device\HarddiskVolume1\WINDOWS\system32\drivers\etc\hosts
0x00000000024a7a90      1      0 R--rwd \Device\HarddiskVolume1\WINDOWS\system32\svchost.exe

$ ./volatility-2.5.standalone.exe -f forensic_100.raw --profile=WinXPSP2x86 dumpfiles -D dumpfiles -Q 0x000000000217b748
Volatility Foundation Volatility Framework 2.5
DataSectionObject 0x0217b748   None   \Device\HarddiskVolume1\WINDOWS\system32\drivers\etc\hosts
```

抽出されたhostsファイルを見る

```
153.127.200.178    crattack.tistory.com 
```

あとは、後は先ほどのドメイン部分を変更して、アクセスする
* http://153.127.200.178/entry/Data-Science-import-pandas-as-pd

```
SECCON{_h3110_w3_h4ve_fun_w4rg4m3_}
```

# 参考サイト
* [「Volatility Frameworkを使ったメモリフォレンジック」と言うハンズオンに参加させて頂きました。](http://dev.classmethod.jp/study_meeting/volatility-framework/)
* [Volatility Command Reference](https://github.com/volatilityfoundation/volatility/wiki/Command%20Reference)
