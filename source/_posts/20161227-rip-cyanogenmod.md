---
title: CyanogenModの終焉とそれから
date: 2016-12-27 00:30:00
index_img: /images/banner-rip-cyanogenmod.png
summary: これまでAndroidをベースとしたアフターマーケットファームウェアの代表格であったCyanogenModは、2016年12月に予期されぬ形で唐突に終わりを迎えることになりました。
---

これまでAndroidをベースとしたアフターマーケットファームウェアの代表格であったCyanogenModは、2016年12月に予期されぬ形で唐突に終わりを迎えることになりました。最初のCyanogenMod 3.1のリリースが2009/07/01とされているので、開発期間も合わせて8年ほどの命ってとこですかね。

<!-- more -->

わたしがこの方面に足突っ込み始めたのは2014年と遅めで、それ以前の事象については残念ながら伝聞でしか知らないので正確な年数を確認することは叶いませんでしたが、後述するCyanogenModのブログ記事で8年って言ってるので8年なのでしょう。


## 単語について

この記事で取り扱っている内容では複数の類似した単語が登場し、混乱を招く元となっているため、以下のような表記を用います。

正式名称 | 記事中の表記 | 概要 | リンク
------------|------------------|--------|----------
cyanogen | Kondik | Steve Kondik。cyanogenはハンドルネーム。CyanogenModの生みの親。 | https://twitter.com/cyanogen
CyanogenMod | CM | Kondikらによるコミュニティ主導で開発されたカスタムROM。 | https://www.cyanogenmod.org
Cyanogen Inc. | cyngn | Kondikによって創設された会社。 | https://cyngn.com
Cyanogen OS | CyOS | cyngnによる商用Android OS。中価格帯スマートフォンに採用された。 | https://cyngn.com/cyanogen-os

なおCyOSの略称は公式に用いられてた覚えはないので、便宜上です。

## できればいつもの記事くらい簡潔に済ませてほしい人向け

- 12/23: cyngnはCMを事実上手放した。CMのnightlyは年内に終了する
    - 12/26に終了するcommitが入りました。12/25までで終わりです https://github.com/CyanogenMod/hudson/commit/a46504638f2276664a9da19c570dd43dcc75efe2
- CyOSも終わりかな
- cyngnは今後もソースは公開されるとしているが、CMコミュニティは今後の開発を継続しない
- 会社としてのcyngnは存続する。閉鎖ではない
- 12/25: CMコミュニティは **LineageOS** を立ち上げ移行すると宣言 https://github.com/lineageos
- 12/26: CMのドメインが早くもアクセス不能に
{% twitter https://twitter.com/CyanogenMod/status/813086249506349056 %}
    - ドメインの有効期限はだいぶ先（来年10月）なので、cyngn側による遮断の可能性…？（未確定）
- 12/27-28: LineageOSのGerrit Code Reviewに移行が完了、今後の開発はLineageにシフト

## もう少し読みたい人向け

### CMの死（cyngnによる支援の終了）

https://web-beta.archive.org/web/20170306045328/https://cyngn.com/blog/cyanogen-services-shutting-down

一連の騒動は2016/12/23、cyngnが突然非常に短い記事を公開したことにより沸き上がりました。

> As part of the ongoing consolidation of Cyanogen, all services and Cyanogen-supported nightly builds will be discontinued no later than 12/31/16. The open source project and source code will remain available for anyone who wants to build CyanogenMod personally.

もうこの時点で単語が紛らわしいものの連続でかなりの混乱を招きました。適当に文脈を補いながら訳すと以下

> 現在進められているcyngnの再編の一環として、（cyngnが提供している）全てのサービスとcyngnによってサポートされていた（CMの）Nightlyビルドは12/31までに終了します。オープンソースプロジェクトとそのソースコードについては個人的にCMをビルドしたい人のために引き続き利用可能になる予定です。

開発はコミュニティベースで進んでいたとはいえ、フルタイムの開発者を抱え、ドメインとサーバ群を所持していたのはcyngnであり、彼らは年末を持ってCMを事実上手放すつもりのようです。毎日あれだけの機種数に向けてビルド実行して配布するサーバ群を保持していくのは厳しいものが有ったのでしょうか。

### Lineage - 	血統、系統、家柄…？

- https://github.com/LineageOS
- http://www.androidpolice.com/2016/12/12/lineageos-seems-very-likely-to-be-cyanogenmods-new-name-at-least-for-now/

話は少し遡り、11月末にcyngnからKondikが退社するという、本来の表記使うと非常にややこしくなる事態が起きた後、CMがLineageOSに改名(rebrand)するのではないかという噂が12/5～13頃に上がりました。GitHubに作られたこのOrganizationは、CMの全てのリポジトリを同期し始めていたのです。また、12/5に lineageos.org と lineageos.com のドメインが取得されていたこともわたしが確認しました。

{% twitter https://twitter.com/lindwurm/status/811358707262984192 %}
{% twitter https://twitter.com/lindwurm/status/811849758759272450 %}

しばらくは皆半信半疑だったのですが、CMのGerrit Code ReviewにあってGitHubに無いcommitがLineageOSに入っていたり、cm-gerritがCMとLineageOSに同時にpushしていることが判明するなど、次第に疑惑は確信へと近づいていきました。

### CMからのさよなら

ほんの束の間のさよなら

https://plus.google.com/u/0/+CyanogenMod/posts/RYBfQ9rTjEH

ブログ含め cyanogenmod.org ドメインのDNS死んでるので急遽Google+で再投稿されました。以下に全文ではありませんが大体の訳。

- インフラ面の廃止に加え、cyngnによってブランドが第三者に売却される懸念が有りました。
- たとえコミュニティがCMを再建し、独自にインフラを確保して再構築しても、CMの継続的な開発はcyngnによるブランドの売却の脅威に頭を悩ませながらになってしまいます。
- それから、 "Cyanogen" という名前について回る汚名があります。
- これを読んでいる皆さんの多くはCMとCyOSの区別を明確にできる支持者であると思われますが、cyngnによる多くのPR活動の汚点をCMから切り離すのは難しいことです。
- CMが金銭的なサポートとソースの共有においてどれだけcyngnに依存していたかを考えると、なぜ混乱が残っているかを理解するのはそう難しくはありません。
- この最新のcyngnによる行動が間違いなくCMにとって致命的な打撃であることは驚くまでもなく明らかなことです。
- しかしながら、CMは常に名前以上の存在であり、インフラ以上の存在でした。
- CMは、Kondikが自宅で始めたときから今まで、過去数千人にも及ぶ個人の貢献者たちの精神、創意工夫、努力に基づいて成功を収めてきました。
- この精神を受け継ぎ、私たち開発者、デザイナ、デバイスメンテナ、翻訳者からなるコミュニティはCMのソースコードと保留中のパッチのフォーク(fork)を作成するために必要な手順を実行しました。
- これは単純な "リブランド(rebrand)" 以上のものです。
- このフォークはCMを定義してきた草の根的なコミュニティ活動に帰ってきましたが、最近のプロフェッショナルな品質と信頼性を維持しています。
- 私たちはLineage(系統)を誇りとし、前進をし続け、伝統を継承し続けていきます。
- ありがとう、そしてさようなら。 CyanogenModチーム

WayBackMachine: https://web.archive.org/web/20161225144318/https://www.cyanogenmod.org/blog/a-fork-in-the-road

### 私たちはここにいます

ここには夢がちゃんとある

[Yes, this is us. - LineageOS - LineageOS Android Distribution](https://www.lineageos.org/Yes-this-is-us/)

lineageos.org のトップから見れる記事一覧には "We aren't going anywhere." って書いてますね。

> そう、これが私たちです。LineageOSはCyanogenModであったものの続きになります。一企業がオープンソースプロジェクトのサポートをやめたからと言って、それが死ななければならないというわけではありません。

前述のCMコミュニティによる記事を踏まえて、CMの系統を継ぐものということでLineageOSがここに誕生しました。

~~火曜日により詳しい情報が出るようなので落ち着いて待ちましょう（書いてる間に火曜になりました…出たら追記します）。~~ Gerritが動き始めた報告がRedditに上がっただけでした。

## 所感

- cyngnは記事中でソースコードは引き続き利用可能としていますが、そりゃGitHubに上げてんだからお前らがわざわざ消しでもしない限りそうなるだろうという感じですね。
- LineageOSの今後の予想についてはXDAの記事読むと良いと思います。考えられる3パターンのシナリオが書かれてます
    - [The Death of CyanogenMod and What it Means for Development](https://www.xda-developers.com/the-death-of-cyangenmod-and-whats-in-store-for-the-future/)
- CMの主要開発者でありLineage Team MemberっつってるHaggertkが[Redditに書いてる](https://redd.it/5ka6zo)のを見ると、DNSとGerrit死んでるのはやっぱりCMとLineageの記事に対するcyngn側の反応なんじゃないかなあという感じです。
    - レビューのコメントを含むGerritのすべての変更などを同期させるのを急いでるということで、火曜までに済ませたいようです
        - 無事にGerritのimportが済み、開発が移行されました。
- わたしはLineageに対して悲観はしていませんし、これからに期待していますし、できる形での貢献はしていきたいと思っています。
- これを読んでいる皆さんもできることからやってみませんかとか。CMがそうであったように、Lineageを良くしていけるのは個々人の貢献の集まりに依るものです。
    - コード書いて本体の開発に参加するもよし
    - 手持ちの端末に移植してデバイスメンテナになるもよし
    - インターフェースの翻訳に参加するもよし
        - どこで使われているかを把握できていれば大半は単語単位なので難しいことはありません
        - ~~(まだCrowdin立ってないけど)~~
        - Crowdin: https://crowdin.com/project/lineageos
    - 時間を割いて手を動かす余裕が無いならDonationも喜ばれます
        - ~~(まだ受付先無いけど)~~
        - [現在のサーバ維持コスト](https://wiki.lineageos.org/costs/) からPaypalとBitcoinでのdonationができます
    - バグ報告もログをきちんと添えてあげると開発者が助かります
- これを読んでいるあなたにとって、2017年がcontribute元年になることを期待して一旦〆ます。

## 追記

上の記事は移行までの慌ただしい流れをリアルタイムで追ってた目線からのものでしたが、他に必要そうな情報を適当に。

### CyanogenMod はどこでダウンロードできる？

LineageOS 13.0/14.1でお使いの機種がサポートされてる場合は https://download.lineageos.org/ をご利用ください。あえてセキュリティパッチレベル古いものを使う理由は特段無いかと思われます。

CyanogenMod 12.1 以前が何らかの理由でどうしても必要な場合は以下。

- Archive.org では Nightly/Snapshot の最後の版がBitTorrentとHTTP(個別ファイル)で提供されています。
    - https://archive.org/download/cmarchive_nighlies
    - https://archive.org/download/cmarchive_snapshots
- itvends.com ではCyanogenModのミラーサーバそのもののバックアップから得られたファイル一式がBitTorrentで提供されています。
    - https://itvends.com/android/

BitTorrentクライアントでのダウンロード時にファイルを選択可能なはずなのでごあんしんください  
[f:id:mordiford:20170408164920p:plain

### CyanogenMod をビルドしたい

LineageOS を使って下さい: https://github.com/LineageOS

- 13.0/14.1 は移行後もメンテされてるので確実に推奨
- それ以前の全リポジトリとブランチも残ってるし11や12.1あたりは稀に手が加えられることもあるので
    - (10.x以前は流石に今更ビルドする人いるのか…？)

ところで: ビルド方法はこちらの記事が便利です（ダイレクトマーケティング）

[LineageOS/CM14.1 のビルド方法 | dev:mordiford](http://dev.maud.io/entry/howto-build-lineageos-14)
