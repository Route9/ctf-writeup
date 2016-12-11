問題
===
    VoIP
    Extract a voice.
    The flag format is SECCON{[A-Z0-9]}.

解
====
問題の[voip.pcap](./voip.pcap)には音声データが入っている。
WiresharkでVoIPの音声を再生する。
あとはただのリスニング問題。
答えは`SECCON{9001IVR}`。

Wiresharkでの音声再生手順は、このサイトの手順の通り。(手抜き...)
http://www.terilogy.com/momentum/wireshark/wireshark01.html
