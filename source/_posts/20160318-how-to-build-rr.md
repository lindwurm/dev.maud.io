---
title: ResurrectionRemix-M のビルド方法
date: 2016-03-18 16:00:00
cover_img: /images/cm3d2-3.jpg
summary: 現在の Resurrection Remix のビルド手順を以下に示します。
---

現在の Resurrection Remix のビルド手順を以下に示します。
<!-- more -->

CyanogenModやその派生でも、概ね似たような手順でビルドできるでしょう。
CyanogenModのビルドについては以下の記事を参照ください。

http://dev.maud.io/entry/2016/04/25/how-to-build-cm13

## 0. はじめに

### 0-1. 分岐について

- この記事では、各見出しに英数字を振っています。数字が0から順路を示し、英字は分岐を示します。
    - 例えば1から2、そして2-aときたら次は3、といった形です。2-bはそっくり飛ばします。お分かりですね？

### 0-2. 検証環境について

#### 0-2-a. Ubuntu 14.04

- 自宅に設置している `mashiro` で検証しました。
    - arm64向けだと初回50分前後、ccacheが効くと30分くらいでビルドできてます

パーツ | 構成 | メーカー | 備考
:-----:|:-----|:---------|:----
CPU | i7-2600K @ 3.40GHz | Intel | 4コア/8スレッド。-j8 で通りますね
RAM | 32GB (`W3U1600PS-8G`) | Panram | 8GB*4。現状積める最大。
SSD0 | 512GB (`TS512GSSD370S`) | Transcend | システムのインストール先及び `/home` 。
SSD1 | 240GB (`SDSSDHII240G`) | SanDisk | `/ssd1`  。作業用環境。
HDD | 3TB (`WD30EZRZ`) | Western Digital | WD青。物置。 `/hdd1` 。
OS | Ubuntu 14.04.5 | - | Server版

- が、当記事ではビルド用のソースディレクトリは `~/rr` として進めます
- 今回のビルドターゲットは Nexus 5 、`hammerhead` です。適宜読み替えてください。

#### 0-2-b. Ubuntu 16.04

- さくらのクラウドで検証しています。
    - これで30分くらいです

パーツ | 構成 | 備考
:-----:|:-----|:----
CPU | 仮想20コア | Xeon E5-2650 v3 @ 2.30GHz
RAM | 64GB | CPUと合わせて 315円/h
OS | [Public] Ubuntu Server 16.04.1 LTS 64bit | アーカイブ(簡単)を使用
ストレージ | SSDプラン/250GB | 46円/h。100GBは結構無理がある

- この構成で 時割: 361円、日割: 3,626円、月額: 72,522円
    - まあ慣れれば環境構築含めても2時間で十分余るくらいなので
- アーカイブ(簡単)使うと管理用ユーザが最初からあるのでそのまま始められて便利ですね

## 1. ビルド環境 (ハードウェア)

ResurrectionRemix/ResurrectedScripts を参考にしています

https://github.com/ResurrectionRemix/ResurrectedScripts/blob/marshmallow/README.md

各構成 | 必要とされる要件
:------|:--------------
CPU | 64bit対応で2コア以上
メモリ | 4GB以上、少なくともコア数*2
ストレージ | 150GB以上の空き
インターネット | まともな速度と安定したもの、良いに越したことは無い。20GB程ダウンロードする必要があるのに注意
OS | x64なLinux環境。この記事ではUbuntu 14.04または16.04を推奨します


各工程 | 見込み所要時間
:------|:--------------
パッケージ導入 | 1時間以内
ダウンロード | インターネット接続環境による。100Mbps以上なら40分以内、5Mbpsなら6-8時間程度?
ROMのビルド | PCのスペックによる

- もしビルド時間を短縮するためにPCのアップグレードを考えているならば、以下の点について考慮しましょう:
    - 強いCPU、SSDか回転数の高いHDD、より多いRAM、より高速なインターネット回線

## 2. ビルド環境のセットアップ (ソフトウェア)

### 2-a. ResurrectedScripts を使用する (個人的には非推奨)

https://github.com/ResurrectionRemix/ResurrectedScripts

これは Resurrection Remix のビルド用スクリプトです。現在は Marshmallow のみがサポートされています。

- 次の手順だけで以下の工程が完了します:
    - パッケージのインストール
    - `repo` のセットアップ
    - RRのソースコード取得

cloneして実行するだけ！お手軽ですね！

```
git clone https://github.com/ResurrectionRemix/ResurrectedScripts.git -b marshmallow scripts
```
```
cd scripts
```
```
./setup.sh
```

あとは対話型なはずなんで指示に従ってれば以下に示す 2-1 から 2-3 までの工程をやってくれるでしょう。

### 2-b. 手動でセットアップする

こちらの手順は他のカスタムROMのビルドでも応用できるので覚えておいても損はないです。

#### 2-b-1. パッケージのインストール

手元で一通り検証して必要そうなパッケージに絞りました。たぶん大丈夫だと思います。
あと、ビルドするマシンに直接Android端末を接続する機会がある場合は `android-tools-adb` や `phablet-tools` 、`android-tools-fastboot` とかも入れとくと便利です。

#### 2-b-1-a. Ubuntu 14.04

OpenJDK 7を使用します。必要なパッケージは以下の通りです。

```
sudo apt-get update && sudo apt-get install autoconf automake bc bison build-essential curl flex g++ g++-multilib gawk gcc gcc-multilib git-core gnupg gperf lib32ncurses5-dev lib32readline-gplv2-dev lib32z1-dev libc6-dev libesd0-dev libexpat1-dev liblz4-1 liblz4-tool liblzma5 liblzma-dev libncurses5-dev libsdl1.2-dev libwxgtk2.8-dev libxml2 libxml2-utils lzop maven openjdk-7-jdk openjdk-7-jre patch pkg-config pngcrush schedtool squashfs-tools texinfo xsltproc zip zlib1g-dev
```

#### 2-b-1-b. Ubuntu 16.04以降

Ubuntu 16.04 では OpenJDK 7 が公式なリポジトリに含まれなくなりました。PPAの導入が必要です。

> Server版で `add-apt-repository` が最初から利用できない場合は、 `sudo apt update && sudo apt install software-properties-common` をお試しください。

```
sudo add-apt-repository ppa:openjdk-r
```

必要なパッケージは以下の通りです。

```
sudo apt update && sudo apt install autoconf automake bc bison build-essential curl flex g++ g++-multilib gawk gcc gcc-multilib git-core gnupg gperf lib32ncurses5-dev lib32readline6-dev lib32z1-dev libc6-dev libesd0-dev libexpat1-dev liblz4-1 liblz4-tool liblzma5 liblzma-dev libncurses5-dev libsdl1.2-dev libwxgtk3.0-dev libxml2 libxml2-utils lzop maven openjdk-7-jdk openjdk-7-jre patch pkg-config pngcrush schedtool squashfs-tools texinfo xsltproc zip zlib1g-dev
```

> ##### コラム (更新: 2016.10.08)
> Ubuntu 16.04 では OpenJDK 7 が公式リポジトリに含まれなくなりました。また、Android NではOpenJDK 8に移行するなどの事情も重なり、8/12に入った以下のcommitにより、OpenJDK 7/8, Oracle JDK 7/8 の全てでCyanogenMod 13のビルドが可能になったはずでした（このcommitを以って `EXPERIMENTAL_USE_JAVA8` オプションは廃止されました）。
> https://github.com/CyanogenMod/android_build/commit/df03dd15694e16d79c4d49ca3fc65be681a53cdb
> 10/8追記: しかしながら、OpenJDK 7/8の混在環境で影響が出るということで、再びCM13のビルドにはまずOpenJDK 7を使用するrevertが入りました。当面の間Marshmallowが必要であれば、Ubuntu 16.04であってもOpenJDK 7を導入するのが賢明でしょう。
> https://github.com/CyanogenMod/android_build/commit/9f360cd1250cf36633efd4fc88ddf4b5fdbdc4a6

### 2-b-2. `repo` のセットアップ

`repo` は複数のgitリポジトリを一括で管理できるツールです。  
詳細はGoogleによるリファレンス ― [Repo command reference | Android Open Source Project](http://source.android.com/source/using-repo.html) を読みましょう。

#### 2-b-2-1. `git config`

Git使ってる人は省略。これから使う予定のある人は `android` の代わりに表示名とメールアドレスを入力してね。

```
git config --global user.name "android"
```
```
git config --global user.email "android"
```

#### 2-b-2-2. repo の導入

```
mkdir ~/bin
```
```
export PATH=~/bin:$PATH
```
パスを通したらダウンロードして実行権限を与えます。
```
curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
```
```
chmod a+x ~/bin/repo
```

誤って `~/bin` を削除したりしない限り、この工程は初回のみです。

#### 2-b-3. ソースコードのダウンロード

ビルド用のディレクトリを作成し、入ります

```
mkdir rr
```
```
cd rr
```

- `repo` を使って、manifestと呼ばれる「どのリポジトリを取得し、どういったディレクトリ構成で配置するのか」を一覧にした設定ファイルのようなもの(実態はxml)を取得し、設定します。
    - `repo init` でセット(initializing)します。 `-u <URL>` でmanifestのソースを指定し、 `-b <revision>` でmanifestのブランチやリビジョンを指定します。
    - この工程は他のROMにも応用できるので覚えると良いです

```
repo init -u https://github.com/ResurrectionRemix/platform_manifest.git -b marshmallow
```

- 今度はmanifestで指定されたリポジトリを全てダウンロードします
    - `repo sync` はmanifestに記述された全てのリポジトリを最新の状態に同期します。回線の速度とお使いのマシンの使用可能なスレッド数に応じて `-j*`オプションを使用してください。一般的に、100Mbps以上の実効速度であれば `-j8` で問題ないとされます。
        - 他のオプションについては機会があればなんか解説します。待てないひとは `repo sync --help` してください。


```
repo sync -j8 -f --force-sync --no-clone-bundle
```

## 3. ビルドの準備

もちろんRRのソース置いてるディレクトリに居ることが前提ですよ？

### 3-1. 各種端末向けのソースを取得する

ちなみにハードウェアキーのある端末向けにRRをビルドする場合は注意すべきことが有ります。以下の記事で確認ください
http://dev.maud.io/entry/2016/05/29/rr-overlay-hwkeys-pref

#### 3-1-a. CyanogenMod公式サポート済端末の場合

要するに [github.com/CyanogenMod](https://github.com/CyanogenMod) 以下にデバイスのリポジトリがあればの話です。

まずはビルド用のコマンド集である `envsetup.sh` をロードします。

```
. build/envsetup.sh
```

ビルドするターゲット端末のコードネームを確認したら、

```
breakfast <device>
```

だけで必要なリポジトリを取得してくれます。Nexus 5だと `breakfast hammerhead` とか。

#### 3-1-b. 非公式なデバイスツリーなら存在する場合

この場合は、自分でmanifestを書く必要があります。まずはローカルなmanifestのディレクトリとmanifestのxmlを作成しましょう。xmlの名前はお好きにどうぞ。ただし、自分でmanifestを書く際は `roomservice.xml` は避けたほうが良いでしょう。

```
mkdir .repo/local_manifests
```
```
touch .repo/local_manifests/hammerhead.xml
```

後はお好みのエディタでmanifestを書きましょう。以下にテンプレートを示します

https://gist.github.com/lindwurm/431d035d677d79726829

さっと見てだいたい分かって頂ける通り、`name` にリポジトリのパス、 `path` にローカルでのパス配置、 `remote` はソースが置かれているホスト等によって指定する必要があります。

少なくとも、デバイス単体のリポジトリ(この例では `CyanogenMod/android_device_lge_hammerhead` )は最低限指定しておく必要があります。  
他にも、 `cm.dependencies` に必要なカーネルおよび兄弟機等との共有なツリー、ハードウェア周りで別途必要なリポジトリ等が記載されていることが多いので**必ず確認しましょう**。

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

欠点としては、既にCyanogenModが動いている端末から取り出す必要が有ることが挙げられます。

#### 3-2-b. Factory Imageから取り出す

https://developers.google.com/android/nexus/images

主にNexusで使用可能な方法です。Factory Imageを展開して、必要なファイルを再配置します。

Nexus 5に関してはスクリプトを用意しています。参考にしてください: https://gist.github.com/lindwurm/90fc353413ed78c00b32/8d5afc10b02e05870e41fa501462d6437edfcd8e:title]

> 注: Factory Imageの展開に於いて `simg2img` を利用するために以下の手順が必要です:  
> https://gist.github.com/lindwurm/4ece562f897bbe03edda

```
cd device/lge/hammerhead
```
```
wget -nc -q https://gist.githubusercontent.com/lindwurm/90fc353413ed78c00b32/raw/8d5afc10b02e05870e41fa501462d6437edfcd8e/extract-from-factory-image.sh
```
```
chmod +x extract-from-factory-image.sh
```
```
./extract-from-factory-image.sh
```

## 4.ビルド

RRは更新が多いので、初回でもここまでの作業中に何かしら更新されていたりしますし、local_manifestsに手を加えたりしているかもしれません。ビルド前に `repo sync` し直すと良いです。

```
repo sync -j8 -f --force-sync --no-clone-bundle
```

### 4-a. ResurrectedScripts を使用する

[2-a](http://dev.maud.io/entry/2016/03/18/how-to-build-rr#2-a-ResurrectedScripts-を使用する) でResurrectedScriptsを使っていれば、 `build.sh` を叩くと対話型のインターフェースでビルドを始めてくれます。

### 4-b. 直接コマンドを叩く

ビルド用のコマンド集みたいなもの、`envsetup` を読み込んで

```
. build/envsetup.sh
```

いざビルド！

```
brunch <device> 2>&1 | tee rr_$(date '+%Y%m%d_%H-%M-%S').log
```

(ログ残すためにこうしてるだけで、 `brunch <device>` だけでもビルドはできます)

## 5. ビルドが無事に通ったら

- 終了後、ROMの`.zip`ファイルは `out/target/product/<device>` に有ります。忘れずに安全な場所に退避させましょう。
- 4.の工程 + 諸々やってくれる便利なスクリプトをGitHubで公開しました。こいつをベースに自分好みのスクリプトを用意すると楽しくなりますよ。  
  https://github.com/lindwurm/madoka
- ストレージに数十GB～100GB程度の十分な空き容量があり、次回以降のビルドを高速化させたい場合は `ccache` の導入を検討しましょう。ここに書くには狭すぎるけど記事にするには短いのであんまり書く気力が無いです。需要があればやるかもしれません。とりあえずArchWiki見てください。
https://wiki.archlinuxjp.org/index.php/Ccache

## 6. あとがき

- もし詰んだらTwitterで@lindwurmに聞いてもらえると分かる範囲で答えられるかもしれません
    - 移植する方の開発者ではないので…
- この記事では、主に **カスタムROMのビルドに興味があるけど手順がよくわからない** という ~~ニッチな~~ 層を対象としてみました。
    - CyanogenMod Wiki か platform_manifest のREADME読んでればできるだろ！と思わなくもないかも知れませんが、初心者の方が ~~日本語じゃないというだけで障壁に感じてしまったり~~ あれだけでビルド通せるともなかなか思えないような書かれ方だったので重い腰を上げてみた所存です。
    - この記事をきっかけに自分でROMをビルドしてみるひとが増えるといいなーとは思います ~~が、増えたところで特に何もないんですよね…~~
        - 自分の端末を真に所有できた気分にはなれると思います。あと開発追って新機能をいち早く試すとか、月例のセキュリティパッチを速攻で適用できるとかはあるかもしれませんね
        - カーネル欄のビルドしたユーザ/ホスト名に自分の名前出せるの最高に自己満足感あるよな
    - ついでに自分の使ってるROMを翻訳とかしてみるといいんじゃないですか。
        - これも記事書けばいいんかな
            - ぶっちゃけこの記事も一日は掛かったので労力がだな
