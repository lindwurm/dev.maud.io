---
title: ハードウェアキーのある端末向けにResurrection Remix Mをビルドする際の注意点
date: 2016-05-29 02:00:00
index_img: /images/cm3d2-11.jpg
summary: Resurrection RemixではCyanogenMod向けのデバイスツリーをそのまま流用してビルドが可能なことはご存知かと思いますが、たまに独自の箇所がありよく注意する必要があります。その一つがハードウェアキーです。
---

Resurrection RemixではCyanogenMod向けのデバイスツリーをそのまま流用してビルドが可能なことはご存知かと思いますが、たまに独自の箇所がありよく注意する必要があります。その一つがハードウェアキーです。

<!--more-->

Resurrection Remixのビルドについては次の記事を参照ください。

[http://dev.maud.io/embed/2016/03/18/how-to-build-rr](http://dev.maud.io/embed/2016/03/18/how-to-build-rr)

## ハードウェアキーの設定を表示させる

Samsung Galaxyを中心に、ナビゲーションボタンがハードウェアキーやセンサーキーを使用している端末は今も根強く残っています。こういったキー配置のメリットとしては、画面をフルに活用できる(尤も、Android 4.4以降ではImmersiveModeが追加されたのでオンスクリーンキーの端末でも全画面表示は行えるようになりましたが)などが挙げられますね。

Resurrection Remixでは、これらのハードウェアキーのオン/オフ及びバックライトを調整する設定があるのですが、**デフォルトでは表示されません**。そのため、CyanogenModのデバイスツリーをそのまま使用してビルドするとこれらの設定を利用できなくなります。

![](https://cdn-ak.f.st-hatena.com/images/fotolife/m/mordiford/20160529/20160529012222.png) ![](https://cdn-ak.f.st-hatena.com/images/fotolife/m/mordiford/20160601/20160601060323.png)

(1枚目がCyanogenModのツリーそのままでビルドした例、2枚目が対応後。)

### 結論から言うと

デバイスツリーの `overlay/frameworks/base/core/res/res/values/config.xml` に以下を追記してビルドしてください。

```xml
<bool name="config_hwKeysPref">true</bool>
```

### 簡単な解説

ResurrectionRemix/android_frameworks_base にて、このハードウェアキーの設定にあたる `config_hwKeysPref` は `false` となっており、デフォルトでは無効化されています。

https://github.com/ResurrectionRemix/android_frameworks_base/blob/marshmallow/core/res/res/values/rr_config.xml#L26

これは今年の1/31に追加された次のcommitによるものです。

https://github.com/ResurrectionRemix/android_frameworks_base/commit/1a4b68c95a77f1cc62cc21960cd3094067e3412c

コメントに

```
PS: to enable this feature you need to set to true <bool name="config_hwKeysPref">true</bool> in device overlay. Of course this is for devices with hardware and capacitive keys ;)
```

と、ハードウェアキーの設定を有効化するにはデバイスツリー側で `config_hwKeysPref` を `true` にセットする必要がある旨を明記されています。
ここのdevice overlayとはデバイスツリーの `overlay/` 以下のことで、フレームワークやアプリの予め設定された値を、その端末に限り値を上書きするためのものです(と認識していますが、間違っていたら教えて下さい)。

例えば、今回のように `frameworks/base/core/res/res/values/rr_config.xml` の値を変えたければ、 `device/[vendor]/[codename]/overlay/frameworks/base/core/res/res/values/config.xml` に上書きしたい値を記述していきます(元の `rr_config.xml` も `config.xml` も、前者がRR独自、後者はCMのを取り込んでるだけなようなのでoverlay側は一纏めに `config.xml` に列挙して問題ないです)。このあたりについてはCyanogenMod Wikiの [Adding XML Overlays - How To Port CyanogenMod To Your Own Device](https://wiki.cyanogenmod.org/w/Doc:_porting_intro#Adding_XML_Overlays) が参考になります。

Resurrection Remixの場合はこのようにしてハードウェアキーの有無を判定し、設定の表示/非表示をコントロールしていたというわけです。

## あとがき

CyanogenModのナビゲーションバーの表示/非表示はRRとはまた違ったやり方だった記憶があるので、時間と気力があればそっちもまとめてみたいと思います…
