---
title: カスタムROMのビルド時にバージョン番号をスマートに取り出す
date: 2016-08-02 06:00:00
---

RR 5.7.2のリリースcommit見たらバージョン番号の取り出し方を覚えたのでメモ。全て8/1時点のものです。
カスタムROMのビルドをスクリプト組んでいろいろしてるひとの参考になれば幸い。

<!-- more -->

## 実例

### CyanogenMod

参照元: https://github.com/CyanogenMod/android_vendor_cm/blob/cm-13.0/config/common.mk

#### 番号のみ

```
$(get_build_var PRODUCT_VERSION_MAJOR).$(get_build_var PRODUCT_VERSION_MINOR)
```

得られる結果: `13.0`

#### zip名

```
get_build_var CM_VERSION
```

得られる結果: `cm-13.0-20160801-UNOFFICIAL-hammerhead`

### MoKee OpenSource

参照元: https://github.com/MoKee/android_vendor_mk/blob/mkm/config/common.mk

#### 番号のみ

```
$(get_build_var PRODUCT_VERSION_MAJOR).$(get_build_var PRODUCT_VERSION_MINOR)
```

得られる結果: `60.1`

#### zip名

```
get_build_var MK_VERSION
```

得られる結果: `MK60.1-hammerhead-201608010000-UNOFFICIAL`

### Resurrection Remix

参照元: https://github.com/ResurrectionRemix/android_vendor_resurrection/blob/marshmallow/config/common.mk

#### 番号のみ

```
get_build_var PRODUCT_VERSION
```

得られる結果: `5.7.2`

#### zip名

```
get_build_var CM_VERSION
```

得られる結果: `ResurrectionRemix-M-v5.7.2-20160801-hammerhead`

### Android Ice Cold Project

参照元: https://github.com/AICP/vendor_aicp/blob/mm6.0/configs/common_versions.mk

#### 番号のみ

```
get_build_var VERSION
```

得られる結果: `11.0`

#### zip名

```
get_build_var AICP_VERSION
```

得られる結果: `aicp_hammerhead_mm-11.0-UNOFFICIAL-20160801`

### HexagonRom

参照元: https://github.com/HexagonRom/android_vendor_hexagon/blob/mm/configs/common_versions.mk

#### 番号のみ

```
get_build_var VERSION
```

得られる結果: `1.3`

#### zip名

```
get_build_var HEXAGON_VERSION
```

得られる結果: `HexagonROM--V1.3-mm-hammerhead-UNOFFICIAL-20160801`

### BlissRoms

参照元: https://github.com/BlissRoms/platform_vendor_bliss/blob/mm6.0/config/versions.mk

#### 番号のみ

```
get_build_var VERSION
```

得られる結果: `6.4`

#### zip名

```
get_build_var BLISS_VERSION
```

得られる結果: `Bliss-v6.4-hammerhead-UNOFFICIAL-2016-0801-0000`

## あとがき

きっかけはこのcommitでした。

https://github.com/ResurrectionRemix/android_vendor_resurrection/commit/88b437052e405ed292bc05ea072d4c97a436b036

`get_build_var` で取れる値はいい感じに覚えておくとバージョン番号取り出すときにcutしたりスクリプトに番号ベタ書きする必要が無くなって便利です。活用しましょう。
