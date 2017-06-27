# 概要
zipを解凍して中のパケットから暗号を解読する問題

# 手順
## 1. Zipを復元
DLしたファイルを解凍するとfile_1 file_2ができるが、zipinfoで見るとfile_3まである  

```
$ zipinfo Forensic_Encyption
Archive:  Forensic_Encyption
Zip file size: 30199 bytes, number of entries: 3
-rw----     2.0 fat    31112 b- defN 17-May-15 16:39 file_3
-rw----     2.0 fat    20874 b- defN 17-May-01 12:40 file_1
-rw----     2.0 fat      418 b- defN 17-May-15 17:10 file_2
3 files, 52404 bytes uncompressed, 29913 bytes compressed:  42.9%
```

どうもfile_3のlocalfileheaderが意図的に壊してあるようなので、復元してやる  
ファイルの先頭2バイトだけ書き換わっているようなので、50 4Bに修正するとfile_3まで復元できた  

## 2. file_2解凍
file_2がパス付きzipになっているのでパスを探す  
file_1のjpegのexifを見ると怪しいコメントが入っている  
```
User Comment : "VHVyaW5nX01hY2hpbmVfYXV0b21hdG9u"
```
Base64デコードすると`"Turing_Machine_automaton"`となり、これがfile_2のパスワード  
file_2の中にはkey.txtが入っている  
```
src 192.168.30.211 dst 192.168.30.251
        proto esp spi 0xc300fae7 reqid 1 mode transport
        replay-window 32
        auth hmac(sha1) 0x2f279b853294aad4547d5773e5108de7717f5284
        enc cbc(aes) 0x9d1d2cfa9fa8be81f3e735090c7bd272
        sel src 192.168.30.211/32 dst 192.168.30.251/32
src 192.168.30.251 dst 192.168.30.211
        proto esp spi 0xce66f4fa reqid 1 mode transport
        replay-window 32
        auth hmac(sha1) 0x3bf9c1a31f707731a762ea45a85e21a2192797a3
        enc cbc(aes) 0x886f7e33d21c79ea5bac61e3e17c0422
        sel src 192.168.30.251/32 dst 192.168.30.211/32
```

## 3. file_3を復号して通信を見る
key.txtの情報を元に、IPSec-ESPのパケットを復号すると、以下の情報を取得しているTCP通信が見える  
```
M4 Navy
Reflector:C Thin, beta, I, IV, II (T M J F), Plugboard: L-X/A-C/B-Y

TMCTF{APZTQQHYCKDLQZRG}

APZTQQHYCKDLQZRG is encrypted.
```

## 4. エニグマ暗号解読
以下のサイトから、file_3の通信内容を元に`APZTQQHYCKDLQZRG`を復号してやる
* [http://enigma.louisedade.co.uk/enigma.html](http://enigma.louisedade.co.uk/enigma.html)

# Flag
`TMCTF{RISINGSUNANDMOON}`
