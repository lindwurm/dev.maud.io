---
title: ビルド用サーバを設置した
date: 2016-01-11 05:00:00
index_img: https://pbs.twimg.com/media/CYRraF1U0AAzydg.jpg
summary: ビールサーバーではないです。
---

ビールサーバーではないです。

<!-- more -->

{% twitter https://twitter.com/lindwurm/status/685776089331793920 %}

設置した、と言っても上のツイートの通り私が組んだわけじゃなく、@ahiru3netさんからopencoconの開発者である@shimadahさんが譲り受けてビルド用マシンになってたやつを、10月にNURO光引いたことで上下共に安定して速いインターネット接続を得られるほたハイツに置いて、わたしもAndroidのビルドに使わせてもらうという感じです。

つよい回線の図(少なくとも上下650Mは確実に出る、平均800M台、最大950Mまで観測)

{% twitter https://twitter.com/lindwurm/status/686103594089590784 %}

![](https://www.speedtest.net/result/6814797264.png)

設置。あの箱には横倒しになって入ってたらしい。でけえ。

{% twitter https://twitter.com/lindwurm/status/685791139551555584 %}

ホスト名は黒猫ダンジョンネタが早くも切れてしまった(登場用語が少ない)ので、適当に好きなキャラ名を@shimadahさんに候補として投げた結果 `mashiro` になりました。

ちなみに元ネタは[明日からテレビアニメも開始](http://aokana-anime.com/)される蒼の彼方のフォーリズムから、ゲーマーな小動物系後輩(かわいい)の [有坂真白](http://aokana.net/character/mashiro/)です。

[![](https://2.bp.blogspot.com/-ketCgxYteMw/VpKp4-mzL2I/AAAAAAAADCM/KM_om_hQpFU/s1600/Screenshot+from+2016-01-11+03-57-23.png)](http://aokana-anime.com/character/)

転校生なメインヒロインの [倉科明日香](http://aokana.net/character/asuka/) と二番目を争う程度に好きなキャラです。ちなみに一番は **ヒロインじゃないけど人気投票で4人居るヒロインの1人を差し置いて4位を獲得した** という伝説(実話)を持つ、~~自称~~ **窓果ちゃんマジメインヒロイン** な 青柳窓果 です。青髪赤目ですよ青髪赤目！！！そしてCV:若林直美！！！

[![](https://4.bp.blogspot.com/-UfbZSuv7EP0/VpKvrWON0MI/AAAAAAAADCc/qasIMIohfZA/s1600/Screenshot+from+2016-01-11+04-23-14.png)](http://aokana-anime.com/character/)

閑話休題。そんなmashiroサーバですが、arm64なLG G4向けのBlissPop(CyanogenMod 12.1ベースの多機能ROM)を1時間11分でビルド完了する程度につよかったです。

(参考記録: i5-3320MとRAM16GB積んだX230でarmなLG G3向けにビルドして2時間半〜3時間弱)

{% twitter https://twitter.com/lindwurm/status/685927495653134336 %}

SSHで入れるようになったので、実家でも外出先でもSSHクライアントがあればビルド始めさせられますね！
