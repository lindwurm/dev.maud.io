---
title: Android ビルドで学ぶ git-repo 入門
date: 2017-04-08 00:00:00
thumbnail: /images/banner-learn-git-repo.png
---

Androidのビルドを行っていく上で避けては通れない `git-repo` を必要なところから理解していきましょう、というドキュメント風味企画

<!--more-->

## git-repo とは何か

`git-repo` (あるいは単純に `repo`) は Android Open Source Project (AOSP) のような、大量の Git リポジトリの集合体として成り立つプロジェクトを一括で管理するために Google によって作られた、python2 で書かれたツールとそのコマンドです。

他にも、Gerrit 等のコードレビューシステムに変更を送るようなこともできますが、ここではビルドする時に必要な、複数のリポジトリを一括で取得/更新する用途に絞って説明していきます。

## repo のインストール

### require

`python = 2.7.x`

### howto

<pre class="code">mkdir -p ~/bin</pre>
<pre class="code">PATH=~/bin:$PATH</pre>
<pre class="code">curl https://storage.googleapis.com/git-repo-downloads/repo &gt; ~/bin/repo</pre>
<pre class="code">chmod a+x ~/bin/repo</pre>

## repo の基礎/利用可能なコマンド

repo は以下のように用います:

<pre class="code">repo &lt;COMMAND&gt; &lt;OPTIONS&gt;</pre>
> コマンドは全部説明すると長いのでビルドに必要なとこだけ抜粋で

### help

repo がインストールされていれば、最新のドキュメントと利用可能なすべてのコマンドは以下で確認できます:

<pre class="code">repo help</pre>

repo の各コマンドについてのヘルプを参照するには、以下のようにします:

<pre class="code">repo help &lt;COMMAND&gt;</pre>

例えば、 `repo init` についての説明や利用可能なオプションを確認するには以下のようにします(この後 `init` の項でも解説しますが…):

<pre class="code">repo help init</pre>

### version

<pre class="code">repo version</pre>

現在インストールされている repo、repo launcher、git、python、GCC のバージョンを表示します。 `repo --version` も同様。

### selfupdate

<pre class="code">repo selfupdate</pre>

`repo` コマンド自身を最新版に更新します。各種 Linux ディストリのパッケージマネージャから取得できるものはたいてい古いので時々お世話になります。

### init

<pre class="code">repo init -u &lt;URL&gt; [&lt;OPTIONS&gt;]</pre>

`.repo/` ディレクトリを作成し、取得される Git リポジトリの実態はこちらに格納されます。また `.repo/manifest.xml`  も作られますが、これは `.repo/manifests/` へのシンボリックリンクです。

#### init のオプション

- `-u`: manifest のリポジトリが存在する URL を指定します。
- `-m`: リポジトリ内で使用する manifest のファイルを指定します。このオプションが指定されない場合、デフォルトでは `default.xml` が使用されます。
- `-b`: revision もしくは branch を指定します。`git clone` の `-b` オプションと同義。

### sync

<pre class="code">repo sync [&lt;OPTIONS&gt;] [&lt;PROJECT_LIST&gt;]</pre>

新しい変更をダウンロードし、ローカルの環境を更新します。 `repo sync` の実行時に `&lt;PROJECT_LIST&gt;` が与えられていない場合は、全てのプロジェクトを同期します。

- 初めてプロジェクトを sync する場合は、`repo sync` は `git clone` と同義です。
- 既に同期済みのプロジェクトについては、`git remote update &amp;&amp; git rebase origin/&lt;BRANCH&gt;` と同義です。

`repo sync` が成功すれば、指定されたプロジェクト内のコードはリモートリポジトリの最新のコードに同期されています。

#### sync のオプション

> これまたビルドに必要な部分のみ。一覧は `repo help sync` か `repo sync --help` か `repo sync -h` で

- `-j &lt;JOBS&gt;` or `--jobs=&lt;JOBS&gt;`: 同時に sync を実行するジョブ数を `repo sync -j8` のように指定します。ネットワーク環境に合わせて指定してください。未指定の場合は manifest に従って決められた値になります

    - Androidだと `sync-j: "4"` が指定されていることが多いです [https://android.googlesource.com/platform/manifest/+/nougat-mr2-release/default.xml#9](https://android.googlesource.com/platform/manifest/+/nougat-mr2-release/default.xml#9) 。

- `-f` or `--force-broken`: 同期に失敗したプロジェクトがあっても sync を継続します。
- `--force-sync`: ローカルに存在する既存の Git リポジトリをリモートから取得したもので **上書きします** 。ローカルで加えた変更は失われることに注意して下さい。
> 以下は必ずしも必要というわけではないけど、ストレージ使用量を少しでも削りたい人向け？

- `--no-clone-bundle`:  HTTP/HTTPS で `$URL/clone.bundle` を無効化します。
- `-c` or `--current-branch`: manifest で指定されたブランチ/リビジョンだけをサーバから取得します。

## manifest について

manifest は repo でリモートのリポジトリをローカルの作業環境の中でどこのディレクトリに取得/配置するかを定める xml で書かれた設定ファイルです。

### `local_manifests` とは

`repo init` で指定したmanifest以外にローカルで別途追加するリポジトリを定義したmanifestのことです。ファイル名の指定は特に必要なく、`.repo/local_manifests/` 以下に存在しているものは全て読み込まれます。

何もしてない状態で

<pre class="code">breakfast hammerhead</pre>

とかすると `.repo/local_manifests/roomservice.xml` の中にやってくれるんですが、個人的にはここを使用することは推奨しません。ここに追記していくのは尚更良くないです。上書きされることがあるためです。

> device configurationsに含まれている `lineage.dependencies` などの `ROMname.dependencies` は主にroomserviceへの書込に用いられますが、自分でlocal_manifestsを書く場合も参考にして下さい。

`.repo/local_manifests/` に `roomservice.xml` 以外の名前で作成しましょう。

以下に hammerhead.xml の例を示します。

<pre class="code lang-xml"><span class="synComment">&lt;?</span><span class="synType">xml version</span>=<span class="synConstant">&quot;1.0&quot;</span><span class="synType"> encoding</span>=<span class="synConstant">&quot;UTF-8&quot;</span><span class="synComment">?&gt;</span>
<span class="synIdentifier">&lt;manifest&gt;</span>
<span class="synComment">&lt;!-- common --&gt;</span>
<span class="synIdentifier">&lt;project </span><span class="synType">name</span>=<span class="synConstant">&quot;LineageOS/android_device_qcom_common&quot;</span><span class="synIdentifier"> </span><span class="synType">path</span>=<span class="synConstant">&quot;device/qcom/common&quot;</span><span class="synIdentifier"> </span><span class="synType">remote</span>=<span class="synConstant">&quot;github&quot;</span><span class="synIdentifier"> </span><span class="synType">revision</span>=<span class="synConstant">&quot;cm-14.1&quot;</span><span class="synIdentifier"> /&gt;</span>
<span class="synIdentifier">&lt;project </span><span class="synType">name</span>=<span class="synConstant">&quot;LineageOS/android_external_stlport&quot;</span><span class="synIdentifier"> </span><span class="synType">path</span>=<span class="synConstant">&quot;external/stlport&quot;</span><span class="synIdentifier"> </span><span class="synType">remote</span>=<span class="synConstant">&quot;github&quot;</span><span class="synIdentifier"> </span><span class="synType">revision</span>=<span class="synConstant">&quot;cm-14.1&quot;</span><span class="synIdentifier"> /&gt;</span>
<span class="synIdentifier">&lt;project </span><span class="synType">name</span>=<span class="synConstant">&quot;LineageOS/android_packages_resources_devicesettings&quot;</span><span class="synIdentifier"> </span><span class="synType">path</span>=<span class="synConstant">&quot;packages/resources/devicesettings&quot;</span><span class="synIdentifier"> </span><span class="synType">remote</span>=<span class="synConstant">&quot;github&quot;</span><span class="synIdentifier"> </span><span class="synType">revision</span>=<span class="synConstant">&quot;cm-14.1&quot;</span><span class="synIdentifier"> /&gt;</span>
<span class="synComment">&lt;!-- hammerhead --&gt;</span>
<span class="synIdentifier">&lt;project </span><span class="synType">name</span>=<span class="synConstant">&quot;LineageOS/android_device_lge_hammerhead&quot;</span><span class="synIdentifier"> </span><span class="synType">path</span>=<span class="synConstant">&quot;device/lge/hammerhead&quot;</span><span class="synIdentifier"> </span><span class="synType">remote</span>=<span class="synConstant">&quot;github&quot;</span><span class="synIdentifier"> </span><span class="synType">revision</span>=<span class="synConstant">&quot;cm-14.1&quot;</span><span class="synIdentifier"> /&gt;</span>
<span class="synIdentifier">&lt;project </span><span class="synType">name</span>=<span class="synConstant">&quot;LineageOS/android_kernel_lge_hammerhead&quot;</span><span class="synIdentifier"> </span><span class="synType">path</span>=<span class="synConstant">&quot;kernel/lge/hammerhead&quot;</span><span class="synIdentifier"> </span><span class="synType">remote</span>=<span class="synConstant">&quot;github&quot;</span><span class="synIdentifier"> </span><span class="synType">revision</span>=<span class="synConstant">&quot;cm-14.1&quot;</span><span class="synIdentifier"> /&gt;</span>
<span class="synIdentifier">&lt;/manifest&gt;</span>
</pre>

#### manifestの要素

- `name`: ざっくり言うと [https://github.com/](https://github.com/) から後ろの username/repository 部分。
- `path`: ローカルでの配置先パス
- `remote`: 取得元のサーバ。既に他のmanifest等でremoteタグにて定義済のもの
- `revision`: 取得するブランチ/タグ。未定義の場合はremoteタグで定義されたもの

#### おまけ: `remove-project` と remote タグの追加について

既に `default.xml` など既存のmanifestで定義されているものをlocal_manifests側で削除するには、`remove-project` タグを使用します。

BitBucketやGitLabなど、既存のmanifestで未定義のサーバからリポジトリを取得する場合は `remote` タグを用いて定義します。

以下にこの2つを用いて、aarch64 用のgccを[UBER Toolchain](https://github.com/UBERTC/)のものに置き換えるmanifestの例を示します。

<pre class="code lang-xml"><span class="synComment">&lt;?</span><span class="synType">xml version</span>=<span class="synConstant">&quot;1.0&quot;</span><span class="synType"> encoding</span>=<span class="synConstant">&quot;UTF-8&quot;</span><span class="synComment">?&gt;</span>
<span class="synIdentifier">&lt;manifest&gt;</span>
<span class="synComment">&lt;!-- UBERTC --&gt;</span>
<span class="synIdentifier">&lt;remote </span><span class="synType">name</span>=<span class="synConstant">&quot;bucket&quot;</span>
<span class="synIdentifier">          </span><span class="synType">fetch</span>=<span class="synConstant">&quot;https://bitbucket.org&quot;</span>
<span class="synIdentifier">          </span><span class="synType">revision</span>=<span class="synConstant">&quot;master&quot;</span><span class="synIdentifier"> /&gt;</span>
<span class="synIdentifier">&lt;remove-project </span><span class="synType">name</span>=<span class="synConstant">&quot;LineageOS/android_prebuilts_gcc_linux-x86_aarch64_aarch64-linux-android-4.9&quot;</span><span class="synIdentifier"> /&gt;</span>
<span class="synIdentifier">&lt;project </span><span class="synType">name</span>=<span class="synConstant">&quot;DespairFactor/aarch64-linux-android-4.9&quot;</span><span class="synIdentifier"> </span><span class="synType">path</span>=<span class="synConstant">&quot;prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9&quot;</span><span class="synIdentifier"> </span><span class="synType">remote</span>=<span class="synConstant">&quot;bucket&quot;</span><span class="synIdentifier"> /&gt;</span>
<span class="synIdentifier">&lt;/manifest&gt;</span>
</pre>

## 参考

- [Downloading the Source  |  Android Open Source Project](https://source.android.com/source/downloading)
- [Repo command reference  |  Android Open Source Project](https://source.android.com/source/using-repo)
