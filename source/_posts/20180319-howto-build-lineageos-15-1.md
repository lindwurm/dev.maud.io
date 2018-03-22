---
title: LineageOS 15.1 のビルド方法
date: 2018-03-19 17:35:00
thumbnail: /images/banner-howto-build-lineage-15-1.png
---

現在の LineageOS 15.1 のビルド手順を以下に示します。公式ビルドへの採用基準が引き上げられた今、お手持ちの機種向けにビルドしてみるのもいかがでしょう。

<!-- more -->

一度覚えてしまえば、他の派生ROMでもだいたい通用する方法です。

Android 7.1ベースのLineageOS 14.1のビルドについては次の記事を参照ください: [LineageOS/CM14.1 のビルド方法 | dev:mordiford](https://dev.maud.io/entry/2016/12/28/howto-build-lineageos-14/)

## 1. ビルド環境 (ハードウェア)

各構成 | 必要とされる要件
:------|:--------------
CPU | 64bit対応で2コア以上
メモリ | 8GB以上を推奨、少なくともコア数*2GBは欲しいところ
ストレージ | 少なくとも150GB以上の空き
インターネット | まともな速度と安定したもの。数十GBダウンロードする必要があるのに注意
OS | x64なLinux環境。

この記事では Ubuntu 16.04.x Server を例に進めます。Desktop版や、その他のディストリビューションでも概ね問題無いでしょう。

各工程 | 見込み所要時間
:------|:--------------
パッケージ導入 | 1時間以内
ダウンロード | インターネット接続環境による。100Mbps以上なら40分以内、5Mbpsなら6-8時間程度?
ROMのビルド | PCのスペックによる

- もしビルド時間を短縮するためにPCのアップグレードを考えているならば、以下の点について考慮するとよいでしょう。優先度の高い順:
    - よりI/O性能の高いSSDや回転数の高いHDD
    - より多くのRAM
    - より新しく強力なCPU
    - より高速なインターネット回線

### 1-a. mashiro

- 自宅に設置している `mashiro` です。
    - arm64向けだと初回で90-140分くらいですかね。
    - ccache使うと35-40分程度まで縮みます。

パーツ | 構成 | メーカー | 備考
:-----:|:-----|:---------|:----
CPU | i7-2600K @ 3.40GHz | Intel | 4コア/8スレッド。
RAM | 32GB (`W3U1600PS-8G`) | Panram | 8GB*4。現状積める最大。
SSD(SATA)0 | 512GB (`TS512GSSD370S`) | Transcend | システムのインストール先及び `/home`。
SSD(SATA)1 | 500GB (`75E500B/IT`) | Samsung | `/ssd1`。開発環境。
SSD(NVMe) | 400GB (`SSDPEDMW400G4X1`) | Intel | `/nvme0`。ビルド環境と `/tmp`。 [^1]
OS | Ubuntu 16.04.4 | - | 64bit必須。Server版 [^2]

[^1]: SSDとM/BはPCIe 3.0対応なんだけど、CPUだけPCIe 2.0までしか対応して無いのでア  
[^2]: Desktop版やその他のLinuxディストリビューションでもなんとかなるとは思いますが、保証はしません。例えばArch Linuxの場合、GNU makeのバージョンが合わなかったりpython2.7が必要になったりで手間が増えます。

- 当記事ではビルド用のソースディレクトリは `~/lineage` として進めます

## 2. ビルド環境のセットアップ (ソフトウェア)

### 2-1. パッケージのインストール

ビルドするマシンに直接Android端末を接続する機会がある場合は `android-tools-adb` や `phablet-tools` 、`android-tools-fastboot` とかも入れとくと便利です。

#### 2-1-a. Ubuntu 16.04 LTS 以降 (推奨)

必要なパッケージは以下の通りです。

```
sudo apt update && sudo apt install autoconf automake bc bison build-essential curl flex g++ g++-multilib gawk gcc gcc-multilib git-core gnupg gperf imagemagick lib32ncurses5-dev lib32readline6-dev lib32z1-dev libc6-dev libesd0-dev libexpat1-dev liblz4-1 liblz4-tool liblzma5 liblzma-dev libncurses5-dev libsdl1.2-dev libwxgtk3.0-dev libxml2 libxml2-utils lzop maven openjdk-8-jdk openjdk-8-jre patch pkg-config pngcrush python schedtool squashfs-tools texinfo unzip xsltproc zip zlib1g-dev
```

#### 2-1-b. Ubuntu 14.04 LTS もしくは 15.10 以前

**15.10 以前のリリース**: そもそもサポートが切れています。一刻も早くサポート中のバージョンへのアップグレードをしてください。

**14.04 LTS**: OpenJDK 8 が公式リポジトリに含まれていません。かつては ppa:openjdk-r から入手することもできましたが、もはやこのPPA上のOpenJDKは更新されていません。サポート期間は 2019年4月 までですが、これからビルド環境を構築するにはおすすめできない構成です。16.04 LTSへのアップグレードを検討してください。

### 2-2. `repo` のセットアップ

`repo` は複数のgitリポジトリを一括で管理できるツールです。

詳細はGoogleによるリファレンス ― [Repo command reference | Android Open Source Project](http://source.android.com/source/using-repo.html) を読みましょう。

わたしが書いた記事もあるのでどうぞ: [Android ビルドで学ぶ git-repo 入門 | dev:mordiford](https://dev.maud.io/entry/2017/04/08/learn-git-repo/)

#### 2-2-1. `git config`

Git使ってる人は省略。単純にビルド環境にしか使わないなら以下の通りでも問題ありませんが、これからGitを使う予定のある人は `android` の代わりに表示名とメールアドレスを入力しておくとよいです。

```
git config --global user.name "android"
```

```
git config --global user.email "android"
```

#### 2-2-2. repo の導入

```
mkdir ~/bin
```

```
export PATH=~/bin:$PATH
```

パスを通したらダウンロードしてパーミッションを付加します。

```
curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
```

```
chmod a+x ~/bin/repo
```

誤って `~/bin` を削除したりしない限り、この工程は初回のみです。

### 2-3. ソースコードのダウンロード

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
repo init -u https://github.com/LineageOS/android.git -b lineage-15.1
```

- 今度はmanifestで指定されたリポジトリを全てダウンロードします
    - `repo sync` はmanifestに記述された全てのリポジトリを最新の状態に同期します。回線の速度とお使いのマシンの使用可能なスレッド数に応じて `-jN`オプションを使用してください。一般的に、100Mbps以上の実効速度であれば N=8 で問題ないとされます。
    - その他のオプションについては [Android ビルドで学ぶ git-repo 入門 | dev:mordiford](https://dev.maud.io/entry/2017/04/08/learn-git-repo/) に書いたとおりなのでやっぱりこちらもおすすめです。

```
repo sync -j8 -f --force-sync --no-clone-bundle
```

## 3. ビルドの準備

もちろんLineageOSのソース置いてるディレクトリに居ることが前提ですよ？

### 3-1. 各種端末向けのソースを取得する

#### 3-1-a. 公式サポート済端末の場合

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

#### 3-1-b. 公式にサポートされていないがツリーは存在する場合: local_manifests を自分で書く

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
  <project name="LineageOS/android_device_lge_bullhead" path="device/lge/bullhead" remote="github" revision="lineage-15.1" />
  <project name="LineageOS/android_kernel_lge_bullhead" path="kernel/lge/bullhead" remote="github" revision="lineage-15.1" />
</manifest>
```

さっと見てだいたい分かって頂ける通り、`name` にリポジトリのパス、 `path` にローカルでのパス配置、 `remote` はソースが置かれているホスト等、 `revision` に リモートのブランチ名などを指定する必要があります。

少なくとも、デバイス単体のリポジトリ(この例では `LineageOS/android_device_lge_bullhead` )は最低限指定しておく必要があります。  
他にも、 `lineage.dependencies` に必要なカーネルおよび兄弟機等との共有なツリー、ハードウェア周りで別途必要なリポジトリ等が記載されていることが多いので **必ず確認しましょう** 。

manifestの書き方も書いてるのでもっかい貼る: [Android ビルドで学ぶ git-repo 入門 | dev:mordiford](http://dev.maud.io/entry/2017/04/08/learn-git-repo)

### 3-2. `vendor` ディレクトリ

残念ながらカスタムROMは完全にオープンなソースだけで動かすことはできないことが多いです。時には [バイナリブロブ](https://ja.wikipedia.org/wiki/%E3%83%90%E3%82%A4%E3%83%8A%E3%83%AA%E3%83%BB%E3%83%96%E3%83%AD%E3%83%96) のお世話になることも有ります。

多くの場合、これらのファイルは `vendor/メーカー/コードネーム` に配置されます。

#### 3-2-a. 実機から取り出す

ビルドしたいターゲットデバイスをPCに接続し、adbが通る状態にしておいてください。

3-1で取得したデバイス固有のソースがあるディレクトリまで移動します

```
cd device/lge/bullhead
```
ここに `extract-files.sh` があるので、実行すると、`proprietary-files.txt` に記載されたバイナリブロブ共を実機から取り出してソースと一緒に配置してくれます。

既にLineageOSが動いている端末から取り出すのが推奨されます。

`proprietary-files.txt` については次の記事が詳しい: [Working with proprietary blobs | LineageOS Wiki](https://wiki.lineageos.org/proprietary_blobs.html)


#### 3-2-b. Factory Imageから取り出す

[Factory Images for Nexus and Pixel Devices | Google Developers](https://developers.google.com/android/nexus/images)

主にNexusで使用可能な方法です。Factory Imageを展開して、必要なファイルを再配置します。

xda-Developersの [[GUIDE][TUT][WINDOWS/LINUX] How To Extract Nexus Factory Images [EASY METHOD]](https://forum.xda-developers.com/nexus-5x/general/guide-how-to-extract-nexus-factory-image-t3562665) とかが参考になると思います。

## 4.ビルド

ここまでの作業中にLineage側で何かしら更新されていたり、local_manifestsに手を加えたりしているかもしれません。ビルド前に `repo sync` し直すと良いです。

```
repo sync -j8 -c -f --force-sync --no-clone-bundle
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

## 5. ビルド終了

### 5-a. 無事に通った場合

- 終了後、ROMの`.zip`ファイルは `out/target/product/<device>` に有ります。忘れずに安全な場所に退避させましょう。
- 4.の工程 + 諸々やってくれる便利なスクリプトをGitHubで公開しました。こいつをベースに自分好みのスクリプトを用意すると楽しくなりますよ。  
    - https://github.com/lindwurm/madoka
- ストレージに数十GB～100GB程度の十分な空き容量があり、次回以降のビルドを高速化させたい場合は `ccache` の導入を検討しましょう。ここで説明するには長くなりすぎるので、詳しい説明はArchWikiをよく読んでください。
    - https://wiki.archlinux.jp/index.php/Ccache
    - ちなみに30GBも割り当てればだいたい恩恵は受けられます。
    - ccache自体はビルド環境の `prebuilts/misc/linux-x86/ccache/ccache` にあります。

### 5-b. 失敗した場合

保存されたログの **末尾から** `error` で検索かけていくと原因が探りやすくなります。下から400行以内には出てくるんじゃないですかね。

## 6. おまけ

### 6-a. rootが無い！

**LineageOSではデフォルトで `su` を同梱しません**。必要であれば、

- ビルド前に `export WITH_SU=true`
- https://download.lineageos.org/extras から addonsu をダウンロード
- Magiskの導入
- SuperSUの導入

などお好きな方法をどうぞ。

### 6-b. brunch とは？

`brunch` はターゲットを指定する `breakfast` のち利用可能な全コアでビルドする `mka` と同義です。すなわち、`brunch` を叩いた時点で概ね問題のない数の `-jN` が指定されています。

ただし `mka` は `/proc/cpuinfo` を見て決めており、多くの場合において、これはCPUに存在しているコア数（Intel HTTに対応したCPUの場合はスレッド数）の等倍です。

自分で指定してビルドするときは逆に、`brunch` するところを `breakfast ${device} && make -jN` (Nの値の適正値は後述) とするとよいです。

#### 6-b-1. make -jN の適正値を探る

AOSPの公式ドキュメントである [Preparing to Build](https://source.android.com/setup/building#build-the-code) ではハードウェアスレッド数の1-2倍程度を `-jN` 引数に指定したときに最も早くなるとして推奨していますが、これは実のところ搭載しているRAMの容量によります。

コア数 × 2GB 程度のRAMであれば等倍くらいが適しており、それ以上のRAMや十分なI/O性能を持ったストレージを搭載している場合は最大で2倍くらいの値を `-jN` 引数に指定すると、AOSPの想定通りビルド時間の多少の短縮に繋がります。逆に、コア数 × 1GB 程度かそれ以下のRAMしか積んでいないのであれば、 `-jN` 引数に与える値を下げたほうがビルド時間が改善されることも起こり得るでしょう。

## 7. あとがき

もし困ったことがあれば、 Twitter: [@lindwurm](https://twitter.com/lindwurm) か Mastodon: [@hota@mstdn.maud.io](https://mstdn.maud.io/@hota) に聞いてもらえると分かる範囲で答えられるかもしれません（移植する方の開発者ではないので保証はできませんが…）。
