---
title: Android N Preview を Nexus 5 向けにビルドした
date: 2016-03-11 06:30:00
---

- Android N Previewやったーー！
<!-- more -->
    - Nexus 5対象外じゃないすかー！
        - やだーー！！
            - おや、[こんなところ](https://android.googlesource.com/device/lge/hammerhead/+/android-n-preview-1) に `hammerhead` 向けの `android-n-preview-1` ブランチがあるじゃないですか
                - ビルドしよう(錯乱)
                    - AOSPのビルドやったこと無い
                        - でもカスタムROMは毎日のようにビルドしてるし、まあどうにかなるでしょ

- 当ブログではTerminalにおけるプロンプトを表記していないので工程は行ごとにコピペでも問題ないです
    - あと作業は最新の状態である Ubuntu Server 14.04 LTS で行いました

## 環境構築

### パッケージ

- わたしの手元では既にMarshmallowをビルドする環境が整ってたので参考にならない…
    - 列挙しても余計なもの含まれそうなのでここでは省略します
    - [Establishing a Build Environment | Android Open Source Project](https://source.android.com/source/initializing.html) 参照でお願いします

### OpenJDK

- Android Nからは **OpenJDK 8** が必要です。
    - `build/core/main.mk` とか書き換えて回避しようとしましたが結局開始直後にビルドがコケました。

```
sudo add-apt-repository ppa:openjdk-r/ppa
```

```
sudo apt-get update && sudo apt-get install openjdk-8-jdk
```

## リポジトリの取得

- ここはAndroidのビルドしたことある人は飛ばして問題ない工程です。リポジトリの取得に使う `repo` のセットアップです

```
mkdir ~/bin
```

```
curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
```

```
chmod a+x ~/bin/repo
```

```
PATH=~/bin:$PATH
```

- `git config` も済ませておいてください。
- ここからリポジトリの取得。
    - `repo sync` のjオプションはお使いの回線の速度とかに応じて変更してください。
        - 100M以上出るなら8で問題ないです。
- わかるとは思いますがこの先ずっと `aosp-n` ディレクトリで作業します

```
mkdir -p aosp-n
```

```
cd aosp-n
```

```
repo init -u https://android.googlesource.com/platform/manifest -b android-n-preview-1
```

```
repo sync -j8
```

- vendor周りが無いので、 `device/lge/hammerhead/proprietary-blobs.txt` に記載されたファイルを実機かFactory Imageなどから取り出して `vendor/lge/hammerhead` 以下にいい感じに用意しましょう。
    - Factory Imageをダウンロードしていい感じに取り出して配置するスクリプトを用意したのでお使いください
        - スクリプトを実行する上で `simg2img` が必要なので用意します

```
git clone https://github.com/anestisb/android-simg2img.git
```

```
cd android-simg2img
```

```
make
```

```
cp simg2img ~/bin/
```

```
cd ..
```

```
rm -rf android-simg2img
```

- 展開するスクリプトは [gist.github.com/lindwurm/90fc353413ed78c00b32](https://gist.github.com/lindwurm/90fc353413ed78c00b32) に置いてます。

```
cd device/lge/hammerhead
```

```
wget -nc -q https://gist.githubusercontent.com/lindwurm/90fc353413ed78c00b32/raw/e605b7c891e293dccfb047cb3f6b596f06f08b9b/extract-files.sh
```

```
chmod +x extract-files.sh
```

```
./extract-files.sh
```

```
cd ../../../
```

## ビルド

- おまじないでは無いけども。Javaはこっちで切り替えたほうが手軽な気がする。
    - 特にわたしは普段Marshmallowのビルドで `java-7-openjdk-amd64` のほう使ってるので。

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

```
export PATH=$PATH:$JAVA_HOME/bin
```

```
export ANDROID_JAVA_HOME=$JAVA_HOME
```

- `envsetup` してターゲットを指定。
    - 今回はNexus 5なのでコードネームは `hammerhead` 。
        - `user`, `userdebug`, `debug`, `tests` とかありますが今回は `userdebug` で。

```
source build/envsetup.sh
```

```
lunch aosp_hammerhead-userdebug
```

- ビルド。 `make` のjオプションもスレッド数やメモリと相談してください。
    - コケた時に原因探るために、こんな感じでビルドしてログを残したほうが幸せになれる気がします(※1行です)

```
logtime=$(date '+%Y-%m-%d_%H-%M-%S') && make -j8 2>&1 | tee "aosp-n_${logtime}_hammerhead.log"
```

## 実機に導入

* **NexusなんだからみんなFactory Imageの焼き方くらい分かるよね！！！！！**
* fastboot(bootloader)に入って以下を実行
  * もちろん端末のデータが飛びます

```
fastboot devices
```

で接続を確認して

```
fastboot flash boot boot.img
```

```
fastboot flash cache cache.img
```

```
fastboot flash recovery recovery.img
```

```
fastboot flash system system.img
```

```
fastboot flash userdata userdata.img
```

```
fastboot reboot
```

- ダメっぽい(Googleロゴが出たり電源が切れたりを繰り返す)場合は電源と音量-を長押ししてbootloaderに入り、 `fastboot` でFactory Imageでも焼いてからビルドし直すなり諦めるなりしましょう
    - **わたしはこの手順でビルドは通りましたが起動しませんでした。**
        - title: Android N Preview を Nexus 5 向けにビルドした (起動できたとは言っていない)
        - ビルド通るけど起動しないってのが一番悲しいよね！
        - XDAで見た感じkernelでSELinuxのポリシーがセットされてないか何かで読めなくて起動しないらしいので様子見
            - 「最悪N6からぶっこ抜いてportするしかねえな！」という声もある
                - Android N Developer Previewでは<u>バックアップを除く複製、改変、追加、再配布、デコンパイル、リバースエンジニアリング、逆アセンブル、Previewを元にした派生物の作成</u>等が **ライセンス条項で禁止** されています
                    - 詳細は原文 [License Agreement | Android Developers](http://developer.android.com/preview/license.html) の3.4を参照のこと
                    - ライセンスは守りましょう
