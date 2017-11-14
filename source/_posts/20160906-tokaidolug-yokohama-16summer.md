---
title: 東海道眺める横浜の夏 2016 に参加・発表してきた
date: 2016-09-06 01:30:00
thumbnail: https://pbs.twimg.com/media/CrbYoldVYAIrYQP.jpg
---

今回のを週刊の記事に入れてしまうと近況がクソ長くなりそうなのでこれ単体でちょっと書くことにしました

<!--more-->

他の参加者による投稿は以下

[東海道らぐ＠横浜（２回目）を開催しました〜 - あしかの部室](http://a-shika.hateblo.jp/entry/2016/09/05/225920)
[最近のスライド - あっきぃ日誌](http://akkiesoft.hatenablog.jp/entry/20160905/1473040607)
[東海道らぐオフラインミーティング 2016年9月横浜　LT大会: Kapperのブログ　新館](http://kapper1224.sblo.jp/article/176729760.html)

## 他の方の発表について

結構ざっくり感想箇条書きですごめんなさい

### `@hashimom`: あひるに焼かれた話と今後のおーぷん万葉

[あひるに焼かれた話と今後のおーぷん万葉について](https://www.slideshare.net/hashimom/ss-65701016) (slideshare)

- 東海道らぐサーバにLet's Encrypt導入 + 常時SSL化
- Chromium系だと混在コンテンツ扱い食らって緑の鍵マーク出ない
- よく見たらConnpassのリンクがHTTP(HTTPSつけてアクセスしてもリダイレクト)
    - 決済有りの勉強会プラットフォームなのにそれどうなの…
- かな漢字変換「`Genji`」
    - 周辺ツール群が `Kasuga`, `Fujitsubo`, `Aoi`, `Murasaki` とネーミングのこだわりが良いのすき

### `@ftake`: UI向け日本語フォントを考える

[Geeko Blog » UI 向け日本語フォントを考える](http://blog.geeko.jp/ftake/1398)

- OpenSUSEのフォント話
- 源ノ角ゴシックが高DPI環境を想定しているせいで表示がアレな話は初耳だった
    - まあAndroidに採用されてるあたり前者は想像に難くなかったけど
        - 行間の広さでメニューの縦幅が広いアレは(フォント自体にそういう意図はないとは思うけど)Material DesignのゆとりをもったUIぽさが感じられた

### `@Masa_B_Kondo`: 海外イベントの可能性

[海外イベントの可能性を探ろう](https://www.slideshare.net/masatakakondo16/ss-65740355) (slideshare)

- 海外か…なかなか行く機会が無いんですよね
- FOSSAsiaのサイトみてみたら面白そうではある
    - https://fossasia.org/

### `@monochrojazz`: 音ゲーコントローラとLinux Input Subsystem

[音ゲーコントローラとLinux Input Subsystem](https://www.slideshare.net/monochrojazz/linux-input-subsystem) (slideshare)

- 専コンプレミアムモデル初めて実物見た
- ターンテーブルに `js0` (joystick) だと不感地帯があって `event0` (マウスやキーボード用のデバイスファイル) で無事に取れた話
    - 海外サードパーティ製コントローラも基盤によってはマウスのxy軸使ってるみたいな話聞いたことあるし似たような感じなのかな
- 制御にRasPi使ってるのと、使う先がAndroidなのが面白かった(そういやLinuxで動く新しめなBMSプレイヤーあるんすかね今)

### `@furikku_ks09`: Waylandについて

[http://www7b.biglobe.ne.jp/~furi_kurms/StudyInfo.htm](http://www7b.biglobe.ne.jp/~furi_kurms/StudyInfo.htm) (PDF配布有り)

- てっきりSailfish OSの話かと思ってた(最後にあったけど)
- Wayland、名前聞いたこと有ると言ってもせいぜいXに代わる次世代ディスプレイサーバくらいの認識しかなかったのでいろいろ聞けて良かった

### `@lindwurm`: ~~表で言えない~~ AndroidカスタムROM概説

- Sailfish OSからのARM繋がりで。
- 内容は後述。
- 重め+長時間(40分)だったのでこの後休憩に
    - すまん

### `@shimadah`: AllwinnerタブレットのOSを作ってみる（中編）

[AllwinnerタブレットのOSを作ってみる(中編)](https://www.slideshare.net/shimadah/allwinneros-65719676) (slideshare)

- "もしかして10年後はタブレット一つでFLOSSに飛び込んで来る人がくる？"
    - 個人的にはAndroidタブレットはもう言うほど流行らないかそもそもメーカーにやる気がないように見えるんすよね…Nexus後継出ないとかSONYの撤退の噂とかで
- 「AllwinnerやメーカーがGPL違反を繰り返している他、ベンダーの提供するkernelにバックドアが仕込まれているのでは？」→「プリインストールされているAndroidは信用ならない、コミュニティが自ら全てをコントロールできるようにLinux/Mainlineを動かそうと言う意欲が強い」
    - これすき
    - わたしの方面ではそもそもそういうSoC避けるとこあるから最終的にメーカー自らが開発も積極的に主導してるQualcommに偏ってるんだよなあ…
- 当日朝とかに実機で動く様子見せてもらってたんですけどやっぱりSDカード刺すだけでブートできるガバガバさほんと草

### `@_EOF_83_EOF_`: クッソ忙しい我々を襲う脆弱性たち

[クッソ忙しい我々を襲う脆弱性たち - Docs.com](https://docs.com/d/embed/D25192920-8782-5219-2160-002139850211%7eM4f20331c-9a35-2562-07b0-c2b7a27d416c)

MBAのHDMIポートに2年近く気づいてなかったのておくれ

{% twitter https://twitter.com/_EOF_83_EOF_/status/771982340759298048 %}

- 脆弱性について調べてきたっていうので普通に内容振り返る感じかと思ったら実際の現場での対応レポートで面白かった
- 聞きながら、Android周りの脆弱性おもしろエピソードを思い出すなどしていた(直近ではQuadrooterのCheckerアプリがセキュリティパッチレベル見て判断してた事例とか)。
    - ただAndroidのセキュリティネタはわたしがやるには深すぎたんすよね

### `@Akkiesoft`: Raspberry Pi Zeroをじゃぶじゃぶ使っていこうな


- ｱｯﾊｲ卵は浮いていません
- Pimoroniの配信で取り上げられたりコメント拾われたりしたとこほんとすき
- **RasPi Zeroはポンド安の今が買い時！！！**
- リモコンとかRasPi(1B)とか薄い本とか頂きましたありがとうございます

{% twitter https://twitter.com/lindwurm/status/771997581081620480 %}
{% twitter https://twitter.com/lindwurm/status/771987162958921728 %}
{% twitter https://twitter.com/lindwurm/status/771987542220484608 %}
{% twitter https://twitter.com/lindwurm/status/771996240791777280 %}

### `@InabaKazsansan`: テオクレ状態

- あひる焼きプラグインの英語対応
- 海外であひる焼きシールの流通してるの草

### `@Kapper1224`: Windows 10タブレットでUbuntu 16.04を入れてみた

[Windows10タブレットにUbuntu16.04を色々入れてみた　2016年度版　Install Ubuntu16.04 on Windows10 Tablet](https://www.slideshare.net/kapper1224/windows10ubuntu16042016install-ubuntu1604-on-windows10-tablet-63862255) (slideshare)

- Winタブだとデバイスマネージャで一通り見れて楽、なるほど
- ハード的には64bitなのにUEFIだけ32bitとかいう地獄
- Wubiってまだやってたんですね(公式からは落ちたよな)と思ったらforkでUEFI対応してるのか https://github.com/hakuna-m/wubiuefi
- 個人じゃ機材集めるのにも限界があるので手持ちのWinタブのドライバ名を公開して欲しいとのこと
    - そういう知見まとめたとこなかなか無いもんなあ

## 懇親会

{% twitter https://twitter.com/lindwurm/status/772004284888080388 %}

- 懇親会で何種類かお酒を一口ずつ飲むイベント発生した
- やっぱ甘いのしかだめっぽいことがよくわかった

{% twitter https://twitter.com/lindwurm/status/772042616569417728 %}

- まあ終わったらみんなでヨドバすわけですよね
    - んでいつもの

{% twitter https://twitter.com/shimadah/status/772046188895805440 %}

## 自分の発表について

> ここは主に当日参加された方にしか伝わらないんじゃないかな

こんな感じのスライド作ってました

![](https://cdn-ak.f.st-hatena.com/images/fotolife/m/mordiford/20160906/20160906010210.png)

- 内容としては前半でカスタムROMとはなんぞや、AOSPベースのROMについて、派生ROMの登場、あたりをざっくり解説してました
    - 初心者向けと言ったすぐ後に構造の話をしたり、あんまり面白みのあるネタではなかったですね
- 後半はオフレコでアレの話をしました
    - 当初より人数結構多くて、「これほんとに言っちゃって大丈夫かな…」とか、結論的になんかマイナスな印象与えてしまわないかなあとちょっと不安だったりしました
        - が、思ってたより笑いが取れてたり、「面白かった」とか「こういうことも正しく理解した上で付き合っていきたいよね」とかの声を頂けたので安心しました
        - 一応こっちメインで作ったので
- 実は結論を最初に書いてしまってから各要素を加えていった感じになります
    - まあ一番の目的であったひとたちからの理解は得られたのでよし
- たぶん二度と話さないかも
- 参考資料は以下: [20160903_tokaidolug_reference.md](https://gist.github.com/lindwurm/ee3450ef03929373d78ff77400c6f82d)

### 追記: 参考ツイート

{% twitter https://twitter.com/lindwurm/status/773859146038771713 %}

[自由なGNUでないディストリビューション - GNUプロジェクト - フリーソフトウェアファウンデーション](https://www.gnu.org/distros/free-non-gnu-distros.html)

{% twitter https://twitter.com/lindwurm/status/773863738248081408 %}

[Technoethical N150 Mini Wireless USB Adapter for GNU/Linux-libre - Technoethical](https://tehnoetic.com/tehnoetic-wireless-adapter-gnu-linux-libre-tet-n150)

{% twitter https://twitter.com/lindwurm/status/773864553297784833 %}

[完全にフリーなAndroid ROM: ReplicantがJelly Beanに対応](https://jp.linux.com/news/inuxcom-exclusive/413259-lco201402010)

## 今後について

- 今までAndroidネタあんまりやるつもり無かったり、ここ暫く発表の類をしてなくてOSC渋ってたんですが、今回の発表を機にちょっと意欲は出た気がします。
- 10月までにいい感じのネタを育てて、11月のOSC東京で何か話せればなーと思います(まだ「これ面白そうだなー」と思いつつ実行に移せてない案はいくらかある)
