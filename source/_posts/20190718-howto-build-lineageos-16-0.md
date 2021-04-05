---
title: LineageOS 16.0 のビルド方法
date: 2019-07-18 00:35:36
tags:
thumbnail: /images/banner-howto-build-lineage-16-0.png
---

LineageOS 16.0 の日本語でのビルド手順を以下に示します。お手持ちの機種向けにビルドしてみるのもいかがでしょう。

<!-- more -->

## 追記

当記事は2019年に書かれたものであり、現在では古くなっている記述があります。
参考に残しておきますが、今後は https://wiki.maud.io/ja/android/build/prepare を参照してください。

## はじめに

一度覚えてしまえば、FlokoROMなどの派生ROMでもだいたい通用する方法です。

英語が読める人は原文とか読んだほうがいいです。

> https://wiki.lineageos.org/devices/enchilada/build

### 過去バージョンをお探しですか？

Version | LineageOS | 記事リンク
:---|:---|:---|
Android 7.1 | LineageOS/CyanogenMod **14.1** | [LineageOS/CM14.1 のビルド方法](https://dev.maud.io/entry/2016/12/28/howto-build-lineageos-14/)
Android 8.1 | LineageOS **15.1** | [LineageOS 15.1 のビルド方法](https://dev.maud.io/entry/2018/03/19/howto-build-lineageos-15-1/)
Android 9.0 | LineageOS **16.0** | ここ

## ビルド環境 (ハードウェア要件)

各構成 | 必要とされる要件
:------|:--------------
CPU | 64bit対応で2コア以上
メモリ | 8GB以上を推奨、少なくとも コア数×2GB は欲しいところ
ストレージ | 少なくとも150GB以上の空き
OS | x64なLinux環境。
ネット接続 | まともな速度と安定したもの。数十GBダウンロードする必要があるのに注意

### 例: さくらのクラウド

パーツ | 構成 |
:-----:|:----|
CPU | 8コア vCPU (Xeon E5-2650 v3 @2.30GHz) |
RAM | 32GB |
SSD | 250GB |
OS | Ubuntu Server 18.04.2 LTS |
NW | 共用100Mbps |
所要時間 | 初回 128分 / ccache有効時 36分 |

## 工程

この記事では Ubuntu 18.04.x Server を例に進めます。Desktop版や、その他のディストリビューションでも概ね問題無いでしょう。

各工程 | 見込み所要時間
:------|:--------------
パッケージ導入 | 1時間以内
ダウンロード | インターネット接続環境による。<br>100Mbps以上なら40分以内、5Mbpsなら6-8時間程度?
ROMのビルド | PCのスペックによる

- もしビルド時間を短縮するためにPCのアップグレードを考えているならば、以下の点について考慮するとよいでしょう。優先度の高い順:
    - よりI/O性能の高いSSDや回転数の高いHDD
        - NVMeとかPCIe Gen4とか楽しみですね
    - より新しく強力なCPU
        - 個人で16コア32スレッドが手に入る時代
    - より多くのRAM
    - より高速なインターネット回線

- 当記事では便宜上、ビルド用のソースディレクトリは `~/lineage` として進めます

## ビルド環境のセットアップ (ソフトウェア)

### パッケージのインストール

ビルドするマシンに直接Android端末を接続する機会がある場合は `android-tools-adb` や `phablet-tools` 、`android-tools-fastboot` とかも入れとくと便利です。

#### Ubuntu 18.04 LTS (推奨) / 16.04 LTS (可)

必要なパッケージは以下の通りです。

```
sudo apt update && sudo apt install bc bison build-essential ccache curl flex g++-multilib gcc-multilib git gnupg gperf imagemagick lib32ncurses5-dev lib32readline-dev lib32z1-dev liblz4-tool libncurses5-dev libsdl1.2-dev libssl-dev libwxgtk3.0-dev libxml2 libxml2-utils lzop pngcrush repo rsync schedtool squashfs-tools unzip xsltproc zip zlib1g-dev
```

#### 非LTSな最新のリリース (執筆時点: 19.04)

一応の手順通りで良いはずだけど、サポート期間に気をつけてください。

#### サポートの切れたリリース

今すぐ Ubuntu 18.04.x をインストールしてください。

### `repo` コマンドのためのセットアップ

`git-repo` (`repo` コマンド) は複数のgitリポジトリを一括で管理できるツールです。 `apt` などのパッケージ管理システムを用いてインストールするのが簡単でしょう。Ubuntuでのパッケージ名は [`repo`](https://packages.ubuntu.com/ja/bionic/repo) です。

詳細はGoogleによるドキュメント、 [Repo command reference | Android Open Source Project](http://source.android.com/source/using-repo.html) を、実践的な使い方はわたしが日本語で書いた [Android ビルドで学ぶ git-repo 入門 | dev:mordiford](https://dev.maud.io/entry/2017/04/08/learn-git-repo/) を読むと良いです。

#### `git config`

Git使ってる人は省略。単純にビルド環境にしか使わないなら以下の通りでも問題ありませんが、これからGitを使う予定のある人は `android` の代わりに表示名とメールアドレスを入力しておくとよいです。

```
git config --global user.name "android"
```

```
git config --global user.email "android"
```

### ソースコードのダウンロード

ビルド用のディレクトリを作成し、入ります

```
mkdir lineage
```

```
cd lineage
```

- `repo` を使って、manifestと呼ばれる「どのリポジトリを取得し、どういったディレクトリ構成で配置するのか」を一覧にした設定ファイルのようなもの(実態はxml)を取得し、設定します。
    - `repo init` で設定(initializing)します。 `-u <URL>` でmanifestのソースを指定し、 `-b <revision>` でmanifestのブランチやリビジョンを指定します。
    - この工程は他のROMにも応用できるので覚えると良いです

```
repo init -u https://github.com/LineageOS/android.git -b lineage-16.0
```

- 今度はmanifestで指定されたリポジトリを全てダウンロードします
    - `repo sync` はmanifestに記述された全てのリポジトリを最新の状態に同期します。回線の速度とお使いのマシンの使用可能なスレッド数に応じて `-jN`オプションを使用してください。一般的に、100Mbps以上の実効速度であれば N=8 で問題ないとされます。
    - その他のオプションについては [Android ビルドで学ぶ git-repo 入門 | dev:mordiford](https://dev.maud.io/entry/2017/04/08/learn-git-repo/) に書いたとおりなのでやっぱりこちらもおすすめです。

```
repo sync -j8 --force-sync --no-clone-bundle
```

## ビルドの準備

もちろんLineageOSのソース置いてるディレクトリに居ることが前提ですよ？

### 各種端末向けのソースを取得する

#### 公式サポート済端末の場合

要するに [github.com/LineageOS](https://github.com/LineageOS) 以下にデバイスのリポジトリがあればの話です。  
（個人的には公式にサポートされてても local_manifests 書くのをおすすめしたいんですが、手軽にやるならこちら）

まずはビルド用のコマンド集である `envsetup.sh` をロードします。

```
. build/envsetup.sh
```

ビルドするターゲット端末のコードネームを確認したら、

```
breakfast <device>
```

だけで必要なリポジトリを取得してくれます。Nexus 5Xだと `breakfast bullhead` とか。

#### 公式にサポートされていないがツリーは存在する場合: local_manifests を自分で書く

公式にサポートされている端末ではbreakfastするだけでいい感じに `~/lineage/.repo/local_manifests/roomservice.xml` に追加されますが、そうでなければ自分でmanifestを書く必要があります。まずはローカルなmanifestを置くディレクトリとmanifestのxmlを作成しましょう。xmlの名前はお好きにどうぞ。ただし、自分でmanifestを書く際は `roomservice.xml` は避けたほうが良いでしょう。

```
mkdir -p .repo/local_manifests
```

```
touch .repo/local_manifests/bullhead.xml
```

後はお好みのエディタでmanifestを書きましょう。以下にテンプレートを示します

```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <project name="LineageOS/android_device_lge_bullhead" path="device/lge/bullhead" remote="github" revision="lineage-16.0" />
  <project name="LineageOS/android_kernel_lge_bullhead" path="kernel/lge/bullhead" remote="github" revision="lineage-16.0" />
</manifest>
```

さっと見てだいたい分かって頂ける通り、`name` にリポジトリのパス、 `path` にローカルでのパス配置、 `remote` はソースが置かれているホスト等、 `revision` に リモートのブランチ名などを指定する必要があります。

少なくとも、デバイス単体のリポジトリ(この例では `LineageOS/android_device_lge_bullhead` )は最低限指定しておく必要があります。  
他にも、 `lineage.dependencies` に必要なカーネルおよび兄弟機等との共有なツリー、ハードウェア周りで別途必要なリポジトリ等が記載されていることが多いので **必ず確認しましょう** 。

> 宣伝: 合同誌[『茜ちゃんの薄い本』](https://akaneblue.booth.pm/) では筆者による local_manifests の書き方を丁寧にレクチャーする記事が掲載されています。.dependencies の読み解き方など、現代のAndroid OSのビルドはまずここから！ご活用ください。

### `vendor` ディレクトリとバイナリブロブ

残念ながらカスタムROMは完全にオープンなソースだけで動かすことはできないことが多いです。時には [バイナリブロブ](https://ja.wikipedia.org/wiki/%E3%83%90%E3%82%A4%E3%83%8A%E3%83%AA%E3%83%BB%E3%83%96%E3%83%AD%E3%83%96) のお世話になることも有ります。

多くの場合、これらのファイルは `vendor/メーカー/コードネーム` に配置されます。

#### 実機から取り出す

ビルドしたいターゲットデバイスをPCに接続し、adbが通る状態にしておいてください。

3-1で取得したデバイス固有のソースがあるディレクトリまで移動します

```
cd device/lge/bullhead
```
ここに `extract-files.sh` があるので、実行すると、`proprietary-files.txt` に記載されたバイナリブロブ共を実機から取り出してソースと一緒に配置してくれます。

既にLineageOSが動いている端末から取り出すのが推奨されます。

`proprietary-files.txt` については次の記事が詳しい: [Working with proprietary blobs | LineageOS Wiki](https://wiki.lineageos.org/proprietary_blobs.html)


#### Factory Imageから取り出す

[Factory Images for Nexus and Pixel Devices | Google Developers](https://developers.google.com/android/nexus/images)

主にNexusで使用可能な方法です。Factory Imageを展開して、必要なファイルを再配置します。

xda-Developersの [[GUIDE][TUT][WINDOWS/LINUX] How To Extract Nexus Factory Images [EASY METHOD]](https://forum.xda-developers.com/nexus-5x/general/guide-how-to-extract-nexus-factory-image-t3562665) とかが参考になると思います。

## ビルド

ここまでの作業中にLineage側で何かしら更新されていたり、local_manifestsに手を加えたりしているかもしれません。ビルド前に `repo sync` し直すと良いです。

```
repo sync -j8 --force-sync --no-clone-bundle
```

```
export LC_ALL=C.UTF-8
export ALLOW_MISSING_DEPENDENCIES=true
```

ビルド用のコマンド集みたいなもの、`envsetup` を読み込んで

```
. build/envsetup.sh
```

いざビルド！

```
brunch <device> 2>&1 | tee lineage_$(date '+%Y%m%d_%H-%M-%S').log
```

(ログ残すためにこうしてるだけで、 `brunch <device>` だけでもビルドはできます)

## ビルド終了

### 無事に通った場合

- 終了後、ROMの`.zip`ファイルは `out/target/product/<device>` に有ります。忘れずに安全な場所に退避させましょう。
- ビルド部分を一発で諸々やってくれる便利なスクリプトをGitHubで公開しました。こいつをベースに自分好みのスクリプトを用意すると楽しくなりますよ。  
    - https://github.com/lindwurm/madoka
- SSD等の比較的I/O性能の良いストレージを利用しており、また数十GB～100GB程度の十分な空き容量がある場合で、次回以降のビルドを高速化させたいときは `ccache` の導入を検討しましょう。ここで説明するには長くなりすぎるので、詳しい説明はArchWikiをよく読んでください。
    - https://wiki.archlinux.jp/index.php/Ccache
    - ちなみに30GBも割り当てればだいたい恩恵は受けられます。
    - ccache自体はビルド環境の `prebuilts/misc/linux-x86/ccache/ccache` にありました…が、最近のUbuntuではパッケージマネージャから降ってくるので素直にそっちに従うほうが良い？

### 失敗した場合

保存されたログの **末尾から** `error` か `FAILED` あたりで検索かけていくと原因が探りやすくなります。下から400行以内には出てくるんじゃないですかね。

## おまけ

### rootが無い！

**LineageOSではデフォルトで `su` を同梱しません**。必要であれば、

- https://download.lineageos.org/extras から addonsu をダウンロード
- [Magisk](https://github.com/topjohnwu/Magisk/releases) の導入

などお好きな方法をどうぞ。

### brunch とは？

`brunch` はターゲットを指定する `breakfast` のち利用可能な全コアでビルドする `mka` と同義です。すなわち、`brunch` を叩いた時点で一般的な環境では概ね問題のない数の `-jN` が指定されています。

この `mka` は `/proc/cpuinfo` を見て決めており、多くの場合において、これはCPUに存在しているコア数（Intel HTTに対応したCPUの場合はスレッド数）の等倍です。

自分で指定してビルドする必要があるときは逆に、`brunch` するところを `breakfast ${device} && make -jN` とするとよいです。

#### make -jN の適正値

- AOSPの公式ドキュメントである [Preparing to Build](https://source.android.com/setup/building#build-the-code) ではハードウェアスレッド数の1-2倍程度を `-jN` 引数に指定したときに最も早くなるとして推奨しています。
- が、筆者が実際に測った結果を[2018年3月の東海道らぐ横浜](https://tokaidolug.connpass.com/event/78990/)で[紹介しています](https://speakerdeck.com/lindwurm/tokaidolug-yokohama-201803)。結果、
    + おおよそコア数 × 2GB 以上のRAMを確保していれば `-jN` を等倍にするのが最適という結論
    + これを2倍に引き上げてもビルド時間は誤差程度にしか短縮されないので **2倍に増やす意味はない**
    + 逆に、コア数 × 1GB 程度かそれ以下のRAMしか積んでいないのであれば、 `-jN` の値を下げたほうがビルド時間が改善される可能性もある

## あとがき

もし困ったことがあれば、 Twitter: [@lindwurm](https://twitter.com/lindwurm) か Mastodon: [@hota@mstdn.maud.io](https://mstdn.maud.io/@hota) に聞いてもらえると分かる範囲で答えられるかもしれません（移植する方の開発者ではないので保証はできませんが…）。
