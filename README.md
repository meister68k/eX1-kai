# eX1改

2020-09-26 by Meister
2021-01-30 CSCベースバージョン 1/24/2021

## 概要

Common Source Code Project( http://takeda-toshiya.my.coocan.jp/ ) のX1エミュレータ eX1に若干の機能追加を施したものです。

本家が取り込んでくれることを期待し，差分のみ公開します。

## 追加した機能

### Visual Studio 2019対応 (2021-01-30)

機能追加ではありませんが，VC++2019でビルドするために変更しています。

#### union前のtypedefを削除 (警告C5208対策)

Visual Studio 2019 ver16.6で追加された，以下の警告への対処です。

>警告 C5208 typedef 名で使用されている名前のないクラスでは、静的でないデータ メンバー、メンバーの列挙値、メンバー クラス以外のメンバーを宣言できません

参考：
https://docs.microsoft.com/ja-jp/cpp/error-messages/compiler-warnings/c5208?view=msvc-160

無名のunionにtypedefで型名を付けるスタイルだと上記に引っかかるためtypedefを削除してunionに直接型名を付けました。

```
// src\common.h 269行ほか
// 元のコード
typedef union {
} pair16_t;

// 変更後
union pair16_t {
};
```

#### vcpkgのzlibを使用する

vcpkg( https://docs.microsoft.com/ja-jp/cpp/build/vcpkg?view=msvc-160 )でzlibのインストールができるので
自前でソースを抱え込むのをやめました。(添付のzlibをVC++2019でビルドするとx64で引っかかった)

> .vcprojの[構成プロパティ]-[Vcpkg]-[General]-[Use Vcpkg]をtrue  

zlib.hのパスが変わるためマクロ定義で対策

> .vcprojの[構成プロパティ]-[C/C++]-[プリプロセッサ]-[プリプロセッサの定義]にUSE_VCPKGを追加

```
// fileio.cpp 30行
// 元のコード
#if defined(USE_QT)

// 変更後
#if defined(USE_QT) || defined(USE_VCPKG) // zlib installed by VCPKG
```


### BASIC ROMボード CZ-8RB (2020-09-26)

**本家12/6/2020版に取り込まれたため改のソースから削除**

クリーンコンピュータのX1にROM起動のBASICを追加するものです。
2DDのべたイメージ(320KB)をCZ8RB.ROMというファイル名で置いておくとIPLが読み込んで起動します。
正規品のROMの中身はCZ-8CB01の起動ロゴだけが変更されたCZ-8RB01らしいのでCZ-8CB01の起動ディスクから
イメージを作ればほぼ同じものが手に入るでしょう。

BASICにこだわらずS-OSやLSX-Dodgersなど任意のOSをFDDよりも高速に起動するために活用することができます。

拙作の互換IPL開発のために必要となり実装しました。実機を持っていませんがX1センター( http://www.x1center.org/ )や
http://www.retropc.net/mm/x1/CZ-8RB/index.html で調査されている内容を参考にしています。


### 偽スーパーインポーズ (2020-09-26版のみ実装)

**実用性のないネタ機能なので改のソースから削除**

ネタ機能です。エミュレータの背景(黒色)を透かしてスーパーインポーズします。
プログラミングをしながらテレビが見られる！これぞパソコンテレビ X1！(専用ディスプレイ CZ-800D は不要です)

![shot2a](https://user-images.githubusercontent.com/17338071/94328733-cbd19c80-ffef-11ea-9e23-de07fe316f23.jpg)

(NHK+って便利ですね)

![shot2](https://user-images.githubusercontent.com/17338071/94328735-dc821280-ffef-11ea-9b4d-4d051b7ad04b.jpg)

ソースコードの#ifdef USE_SUPERIMPOSE周辺が変更点です。
SetLayeredWindowAttributesで透明色を指定するだけですが，RGB=(0,0,0)を指定するとGUI操作ができなくなる不具合があり黒の描画色をわざわざRGB=(1,1,1)にしています。
