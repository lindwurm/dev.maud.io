---
title: LineageOS/CM14.1 のビルド方法
date: 2016-12-28 15:00:00
cover_img: /images/banner-howto-build-cm14.png
summary: 現在の LineageOS/CM14.1 のビルド手順を以下に示します。Weeklyビルドは始まったとはいえ、まだ機種は限られているので、この機会に自分でビルドしてみるのはいかがでしょうか？
---

現在の LineageOS/CM14.1 のビルド手順を以下に示します。Weeklyビルドは始まったとはいえ、まだ機種は限られているので、この機会に自分でビルドしてみるのはいかがでしょうか？
<!-- more -->

一度覚えてしまえば、他の派生ROMでもだいたい通用する方法です。

CM13のビルドについては次の記事を参照ください: [CyanogenMod 13のビルド方法 | dev:mordiford](http://dev.maud.io/entry/2016/04/25/how-to-build-cm13)

## 0. はじめに

### 0-1. 分岐について

- この記事では、各見出しに英数字を振っています。数字が0から順路を示し、英字は分岐を示します。
    - 例えば1から2、そして2-aときたら次は3、といった形です。2-bはそっくり飛ばします。お分かりですね？

### 0-2. 動作環境について

#### 0-2-a. Ubuntu 14.04

- 自宅に設置している `mashiro` です。
    - arm64向けだと初回1時間半くらいですかね。
    - ccache使うと45分程度です

パーツ | 構成 | メーカー | 備考
:-----:|:-----|:---------|:----
CPU | i7-2600K @ 3.40GHz | Intel | 4コア/8スレッド。-j8 で通りますね
RAM | 32GB (`W3U1600PS-8G`) | Panram | 8GB*4。現状積める最大。
SSD0 | 512GB (`TS512GSSD370S`) | Transcend | システムのインストール先及び `/home` 。
SSD1 | 240GB (`SDSSDHII240G`) | SanDisk | `/ssd1`  。開発環境。
OS | Ubuntu 14.04.5 | - | Server版

- 当記事ではビルド用のソースディレクトリは `~/lineage` として進めます
- 今回のビルドターゲットは Nexus 5 、`hammerhead` です。適宜読み替えてください。

#### 0-2-b. Ubuntu 16.04

- さくらのクラウドを使用しています。
    - これで45分くらいです

パーツ | 構成 | 備考
:-----:|:-----|:----
CPU | 仮想20コア | Xeon E5-2650 v3 @ 2.30GHz
RAM | 64GB |
OS | [Public] Ubuntu Server 16.04.1 LTS 64bit | アーカイブ(簡単)を使用
ストレージ | SSDプラン/250GB | 100GBは無理

- この構成で 時割: 361円、日割: 3,626円、月額: 72,522円
    - まあ慣れれば環境構築含めても2時間で十分余るくらいなので
- アーカイブ(簡単)使うと管理用ユーザが最初からあるのでそのまま始められて便利ですね

#### 0-2-c. Ubuntu 16.10 (おまけ)

- さくらのクラウドで 2017/02/09 検証。
    - これで初回1時間と10分くらい。

パーツ | 構成 | 備考
:-----:|:-----|:----
CPU | 仮想8コア | Xeon E5-2650 v3 @ 2.30GHz
RAM | 32GB |
OS | [Public] Ubuntu Server 16.10 64bit | パブリックISOイメージよりインストール
ストレージ | SSDプラン/250GB | 100GBは無理

- 時割192円、日割1,941円、月額38,826円

## 1. ビルド環境 (ハードウェア)

各構成 | 必要とされる要件
:------|:--------------
CPU | 64bit対応で2コア以上
メモリ | 8GB以上を推奨、少なくともコア数*2GBは欲しいところ
ストレージ | 少なくとも150GB以上の空き
インターネット | まともな速度と安定したもの。数十GBダウンロードする必要があるのに注意
OS | x64なLinux環境。この記事ではUbuntu 14.04または16.04のServer版を例に進めます。  Desktop版や、その他のディストリビューションでも概ね問題無いでしょう。

各工程 | 見込み所要時間
:------|:--------------
パッケージ導入 | 1時間以内
ダウンロード | インターネット接続環境による。100Mbps以上なら40分以内、5Mbpsなら6-8時間程度?
ROMのビルド | PCのスペックによる

- もしビルド時間を短縮するためにPCのアップグレードを考えているならば、以下の点について考慮しましょう:
    - より新しく強力なCPU、よりI/O性能の高いSSDや回転数の高いHDD、より多いRAM、より高速なインターネット回線

## 2. ビルド環境のセットアップ (ソフトウェア)

### 2-1. パッケージのインストール

ビルドするマシンに直接Android端末を接続する機会がある場合は `android-tools-adb` や `phablet-tools` 、`android-tools-fastboot` とかも入れとくと便利です。

#### 2-1-a. Ubuntu 14.04

Ubuntu 14.04 には OpenJDK 8 が公式リポジトリに含まれていません。PPAからの導入がおすすめです。

> Server版で `add-apt-repository` が最初から利用できない場合は、 `sudo apt-get update && sudo apt-get install software-properties-common` をお試しください。

```
sudo add-apt-repository ppa:openjdk-r
```

```
sudo apt-get update && sudo apt-get install autoconf automake bc bison build-essential curl flex g++ g++-multilib gawk gcc gcc-multilib git-core gnupg gperf imagemagick lib32ncurses5-dev lib32readline-gplv2-dev lib32z1-dev libc6-dev libesd0-dev libexpat1-dev liblz4-1 liblz4-tool liblzma5 liblzma-dev libncurses5-dev libsdl1.2-dev libwxgtk2.8-dev libxml2 libxml2-utils lzop maven openjdk-8-jdk openjdk-8-jre patch pkg-config pngcrush schedtool squashfs-tools texinfo xsltproc zip zlib1g-dev
```

#### 2-1-b. Ubuntu 16.04以降

必要なパッケージは以下の通りです。

```
sudo apt update && sudo apt install autoconf automake bc bison build-essential curl flex g++ g++-multilib gawk gcc gcc-multilib git-core gnupg gperf imagemagick lib32ncurses5-dev lib32readline6-dev lib32z1-dev libc6-dev libesd0-dev libexpat1-dev liblz4-1 liblz4-tool liblzma5 liblzma-dev libncurses5-dev libsdl1.2-dev libwxgtk3.0-dev libxml2 libxml2-utils lzop maven openjdk-8-jdk openjdk-8-jre patch pkg-config pngcrush python schedtool squashfs-tools texinfo xsltproc zip zlib1g-dev
```

### 2-2. `repo` のセットアップ

`repo` は複数のgitリポジトリを一括で管理できるツールです。  
詳細はGoogleによるリファレンス ― [Repo command reference | Android Open Source Project](http://source.android.com/source/using-repo.html) を読みましょう。

わたしが書いた記事もあるのでどうぞ: [Android ビルドで学ぶ git-repo 入門 | dev:mordiford](http://dev.maud.io/entry/2017/04/08/learn-git-repo)

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
repo init -u https://github.com/LineageOS/android.git -b cm-14.1
```

- 今度はmanifestで指定されたリポジトリを全てダウンロードします
    - `repo sync` はmanifestに記述された全てのリポジトリを最新の状態に同期します。回線の速度とお使いのマシンの使用可能なスレッド数に応じて `-j*`オプションを使用してください。一般的に、100Mbps以上の実効速度であれば `-j8` で問題ないとされます。
        - 他のオプションについては機会があればなんか解説します。待てないひとは `repo sync --help` してください。


```
repo sync -j8 -f --force-sync --no-clone-bundle
```

## 3. ビルドの準備

もちろんLineageOSのソース置いてるディレクトリに居ることが前提ですよ？

### 3-1. 各種端末向けのソースを取得する

#### 3-1-a. CM公式サポート済端末の場合

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

だけで必要なリポジトリを取得してくれます。Nexus 5だと `breakfast hammerhead` とか。

#### 3-1-b. 公式にサポートされていないがツリーは存在する場合: local_manifests を自分で書く

公式にサポートされている端末ではbreakfastするだけでいい感じに `~/lineage/.repo/local_manifests/roomservice.xml` に追加されますが、そうでなければ自分でmanifestを書く必要があります。まずはローカルなmanifestを置くディレクトリとmanifestのxmlを作成しましょう。xmlの名前はお好きにどうぞ。ただし、自分でmanifestを書く際は `roomservice.xml` は避けたほうが良いでしょう。

```
mkdir -p .repo/local_manifests
```

```
touch .repo/local_manifests/hammerhead.xml
```

後はお好みのエディタでmanifestを書きましょう。以下にテンプレートを示します

{% gist 6f0951e8dc3ce10bfbf880592f5c5f25 hammerhead.xml %}

さっと見てだいたい分かって頂ける通り、`name` にリポジトリのパス、 `path` にローカルでのパス配置、 `remote` はソースが置かれているホスト等、 `revision` に リモートのブランチ名などを指定する必要があります。

少なくとも、デバイス単体のリポジトリ(この例では `LineageOS/android_device_lge_hammerhead` )は最低限指定しておく必要があります。  
他にも、 `lineage.dependencies` に必要なカーネルおよび兄弟機等との共有なツリー、ハードウェア周りで別途必要なリポジトリ等が記載されていることが多いので**必ず確認しましょう**。


manifestの書き方も書いてるのでもっかい貼る: [Android ビルドで学ぶ git-repo 入門 | dev:mordiford](http://dev.maud.io/entry/2017/04/08/learn-git-repo)

### 3-2. `vendor` ディレクトリ

残念ながらカスタムROMは完全にオープンなソースだけで動かすことはできないことが多いです。時にはバイナリブロブのお世話になることも有ります。

多くの場合、これらのファイルは `vendor/メーカー/コードネーム` に配置されます。

#### 3-2-a. 実機から取り出す

ビルドしたいターゲットデバイスをPCに接続し、adbが通る状態にしておいてください。

3-1で取得したデバイス固有のソースがあるディレクトリまで移動します

```
cd device/lge/hammerhead
```
ここに `extract-files.sh` があるので、実行すると、`proprietary-blobs.txt` に記載されたバイナリブロブ共を実機から取り出してソースと一緒に配置してくれます。

既にCyanogenMod/LineageOSが動いている端末から取り出すのが推奨されています。

追伸: `proprietary-files.txt` については以下が詳しい。

[Working with proprietary blobs | LineageOS Wiki](https://wiki.lineageos.org/proprietary_blobs.html)

#### 3-2-b. Factory Imageから取り出す

[Factory Images for Nexus and Pixel Devices | Google Developers](https://developers.google.com/android/nexus/images)

主にNexusで使用可能な方法です。Factory Imageを展開して、必要なファイルを再配置します。

Nexus 5に関してはスクリプトを用意しています。参考にしてください: https://gist.github.com/lindwurm/90fc353413ed78c00b32

> 注: Factory Imageの展開に於いて `simg2img` を利用するために以下の手順が必要です:  
> https://gist.github.com/lindwurm/4ece562f897bbe03edda

```
cd device/lge/hammerhead
```

```
wget -nc -q https://gist.githubusercontent.com/lindwurm/90fc353413ed78c00b32/raw/7afd5d068bc2374a341f050b8bd6ad4b8f323bb9/extract-from-factory-image.sh
```

```
chmod +x extract-from-factory-image.sh
```

```
./extract-from-factory-image.sh
```

## 4.ビルド

ここまでの作業中にLineage側で何かしら更新されていたり、local_manifestsに手を加えたりしているかもしれません。ビルド前に `repo sync` し直すと良いです。

```
repo sync -j8 -f --force-sync --no-clone-bundle
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
    - ~~ついでにcontributeしていってくれ～たのむ～~~
- ストレージに数十GB～100GB程度の十分な空き容量があり、次回以降のビルドを高速化させたい場合は `ccache` の導入を検討しましょう。ここに書くには狭すぎるけど記事にするには短いのであんまり書く気力が無いです。需要があればやるかもしれません。とりあえずArchWiki見てください。
    - https://wiki.archlinuxjp.org/index.php/Ccache

### 5-b. 失敗した場合

保存されたログの **末尾から** `error` で検索かけていくと原因が探りやすくなります。下から400行以内には出てくるんじゃないですかね。

#### 5-b-1. Gelloが原因になっている場合

**Lineageでは1/1、派生ROMでもAICPでは1/2など順次対策のcommitが入っています。他のROMでも現在では必要ないかもしれません。**

https://github.com/LineageOS/android_vendor_cm/commit/8d0369c962f3740ef1536dbda5f40e450be1ea4d

Gelloを https://maven.cyanogenmod.org/artifactory/gello_prebuilds/ から取得するようになっている場合、証明書に起因する問題でTerminal経由でアクセスできない場合があります。  
ここでは一時的な解決策として、Gelloを無効にしてビルドする方法を挙げます。 ~~どうせChromeなりFirefoxなり他のブラウザ入れるでしょ皆さん~~

`device.mk` から

```
# Gello		
 PRODUCT_PACKAGES += \		
     Gello
```

の記述を削除またはコメントアウトしてください。

## 6. あとがき

- **LineageOSではデフォルトで `su` を同梱しません**。必要であれば、ビルド前に `export WITH_SU=true` してください
- もし詰んだら Twitter: @lindwurm か Mastodon: @hota@mstdn.maud.io に聞いてもらえると分かる範囲で答えられるかもしれません
    - 移植する方の開発者ではないので保証はできませんが…
- そういえばCyanogenMod Wikiもアクセス不能になってましたね…
    - 一応Weyback Machineによるバックアップは以下
        - http://web.archive.org/web/20161224192646/https://wiki.cyanogenmod.org/w/Main_Page/ja
- 次のステップとしては「何か自分で追加してみる」あたりですかね。ついでにGit/GitHubの使い方も覚えませんか。
