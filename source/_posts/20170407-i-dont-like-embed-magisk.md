---
title: カスタムROMへのMagiskの同梱が嫌いだという話
date: 2017-04-07 16:00:00
cover_img: /images/banner-embed-magisk.png
summary: タイトル通りです
---

わたしがAICPに関わってるのでAICPのやり方に関してだけ書いていきますが、同様な他ROMも嫌いです。

<!-- more -->

## なんで

- そもそも全員がSystemlessなroot環境を必要としているわけではない
- 同梱だと最新のものに追従が遅れる
    - 事実、`MagiskManager.apk` も `Magisk.zip` も2/11に同梱を始めたcommit以降更新がされていない
      - https://github.com/AICP/vendor_aicp/commits/n7.1/prebuilt/common/app/MagiskManager.apk
      - https://github.com/AICP/vendor_aicp/commits/n7.1/prebuilt/common/magisk.zip
    - あとAICPに同梱してるやつ、謎の `vtest4` 版らしいんだけどなにこれ…
- よく見ると結局同梱している `Magisk.zip` を焼いているだけ
    - https://github.com/AICP/build/commit/2b25066e4ee953ee9d5c4825c1d4b6b156c450dd
    - 各自で焼くからもういいよ…
- （もう取り込まれたっぽいけど）パッチが必要な機種もあり、安易な全機種への同梱は勧められたものではない
    - https://forum.xda-developers.com/crossdevice-dev/sony/patched-magisk-v12-sony-devices-t3582331

> ちなみにMagiskの同梱自体はtopjohnwu氏に許可取ってる（というかOSSだから好きにしてくれみたいな感じだった）らしいので、Google製アプリを平然と同梱しているROMとかみたいな問題は起きていない

## mordiford-AICP での対策

- `vendor_aicp` からMagisk関連を削除
    - https://github.com/mordiford/vendor_aicp/commit/280cf5f13f4bc459cb9bcd2d95978f487949bca8
    - revertじゃなく削除なのでLineageOS同様に標準では `su` 非同梱
        - 各自必要に応じて Magisk/SuperSU/superuser とか焼いて下さい
- `build` はさっき上で挙げた焼いてるだけのcommitをrevert
- `packages_apps_Settings` の設定項目は残す
    - Magiskが無ければ表示されないし、別途焼いた場合は表示されるようになるので

> 同梱やめた後に自分で焼いて試したらいい感じだったので、Magisk自体は嫌いではないです。
