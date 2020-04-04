---
title: WSLでAndroidをビルドする話
date: 2018-03-28 20:37:00
---

タイトル通り、Windows Subsystem for Linux (WSL) で Android OS をビルドします。

<!-- more -->

# tl;dr

2018年3月の時点ではWSLでAndroidをビルドするのはやめておけとしか言いようがない。Windowsマシンしか無いならVMを立てたほうが速い。

# やってみてしまった人の末路

## Ubuntu のインストール

適当にMicrosoft Storeを開いて `ubuntu` とか検索するとなんかWSLの特設ページ行きのバナーが出るので、そこから入れるのが安心だと思います。

![](/images/image-wsl-install-1.png)

![](/images/image-wsl-install-2.png)

あとは指示に従ってユーザを生やしたりパスワードを設定したりしてください。

## 必要なパッケージの取得

[LineageOS 15.1 のビルド方法 | dev:mordiford](https://dev.maud.io/entry/2018/03/19/howto-build-lineageos-15-1/) を参照のこと。

### 付録

めんどい人向け。

```
curl -sL https://raw.githubusercontent.com/lindwurm/Scripts/master/setupAndroidBuildEnv.sh | bash -
```

- `git config --global user.name "android"` および `git config --global user.email "android"` が含まれるので、WSL環境を普通に使うつもりがあるなら面倒臭がらないほうが良い。

## ソースコードの取得

AICPの場合。

```
mkdir aicp && cd aicp
```

```
repo init -u https://github.com/AICP/platform_manifest.git -b o8.1
```

<details><summary> 8/12更新: 以下は現在では不要（クリックして展開）</summary>

```
mkdir -p .repo/local_manifests && nano .repo/local_manifests/wsl.xml
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <!-- for wsl -->
  <remove-project name="platform/prebuilts/misc" />
  <project name="Uldiniad/android_prebuilts_misc" path="prebuilts/misc" />

  <remove-project name="LineageOS/android_prebuilts_build-tools" />
  <project name="Uldiniad/android_prebuilts_build-tools" path="prebuilts/build-tools" />

  <remove-project name="AICP/build" />
  <project path="build/make" name="mordiford/build" groups="pdk" remote="github" revision="o8.1_wsl" >
    <copyfile src="core/root.mk" dest="Makefile" />
    <linkfile src="CleanSpec.mk" dest="build/CleanSpec.mk" />
    <linkfile src="buildspec.mk.default" dest="build/buildspec.mk.default" />
    <linkfile src="core" dest="build/core" />
    <linkfile src="envsetup.sh" dest="build/envsetup.sh" />
    <linkfile src="target" dest="build/target" />
    <linkfile src="tools" dest="build/tools" />
  </project>

</manifest>
```

</details>

あとdeviceの分もお忘れなく。

```
repo sync -j16 -c -f --force-sync --no-clone-bundle
```

### 注意

~~上ではもう mordiford/build にパッチ当ててあるのでそっちに入れ替えているけど、LineageOSやその派生には https://review.lineageos.org/#/c/208102/ を当てる必要があります。LineageOSなら `repopick 208102` で当たるし、派生は右上の [Download▼] からパッチを入手して当てるとよいです。~~

2018/07/28くらいに[一連のパッチがmergeされた](https://github.com/LineageOS/android_build/commits/a82a188b512ff34ae8f7e128d804ccaa56bb8902)のでLinuxでの手順と差異がなくなりました。

## ビルド

```
. build/envsetup.sh
```

```
breakfast <device> && mka bacon WITH_DEXPREOPT=false
```

## 結果

![](/images/image-complete-build-on-wsl.png)

すごい！WSL上でビルドが通る！！というのもそうだけど、寝てる間に走らせといたら **9時間かかった** 。

i7-2700K / 16GB(DDR3) / 500GB SSD みたいなよくある一昔前の構成とはいえ、それでもだいたい直接Ubuntuぶち込んでビルドするときの2-3倍近くはかかる感じ。

どうしてそんなに遅くなるのかについてとか、実際WSLがどういう用途なら適してるのか、みたいな細かい話はsatさんの [WSL vs VM for 開発作業](https://satoru-takeuchi.hatenablog.com/entry/2020/03/26/011540) がその通りだなあという感じなので読んでみると良いと思います。

ネイティブ環境やVM上のLinux環境には敵わないとはいえ、WSLでも一応ビルドが可能になったというのは大変意義深いですね、という話でした。

## 参考資料

- LineageOS Code Review
    - [Change Ibf45c7f6: Adapt ijar for WSL](https://review.lineageos.org/#/c/208102/)
        - 該当するpatch。
    - [Change I27e7995c: Add detection for Microsoft Windows LXSS](https://review.lineageos.org/#/c/208384/)
- xda-developers
    - [[GUIDE] How to build LineageOS 15.1 on Windows 10](https://forum.xda-developers.com/android/software-hacking/guide-how-to-build-lineageos-15-1-t3750175)
        - patch投げてたUldiniad本人による手順解説。パッケージが足りない。
- dev:mordiford
    - [LineageOS 15.1 のビルド方法](https://dev.maud.io/entry/2018/03/19/howto-build-lineageos-15-1/)
        - 自分の記事。パッケージ一覧の参考
- GitHub
    - [Adapt ijar for WSL](https://github.com/LineageOS/android_build/commit/14925d0308495cc0877f52cbdbce3d2f320eaea1)
    - [Add detection for WSL](https://github.com/LineageOS/android_build/commit/ecf613729a0b5f9899182b19fe614abe38f7d18e)
    - [dex2oat: disable multithreading for WSL](https://github.com/LineageOS/android_build/commit/17bc5883d52f25416cc2a9ebcd276c1f3e8f37da)
    - [core: config: Use host ijar if requested](https://github.com/LineageOS/android_build/commit/a82a188b512ff34ae8f7e128d804ccaa56bb8902)
