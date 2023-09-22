# iRIC version 4 用ソルバ開発手順

## 概要

1. iRIC v4 (beta版) を入手する。
2. ソルバのビルド環境を用意する。
3. ソルバを iRIC v4 用に修正、動作確認をする。
4. iRIC インストーラに最新のソルバを登録する環境を作成する。

## iRIC v4 (beta版) の入手

以下からインストーラをダウンロードして実行する。

https://github.com/i-RIC/online_update/raw/master/iRIC_Installer_v4beta.exe

## ソルバのビルド環境の用意

C, C++, FORTRAN で開発されたソルバをビルドする場合に必要。Python で開発されたソルバでは不要。

Visual Studio 2019 と Intel One API をインストールする。

以下のページを参照してインストールする。

https://qiita.com/Kazutake/items/a069f86d21ca43b6c153

上記のほか、iRIC のソルバ開発用 SDK (ライブラリ、ヘッダファイル) が必要となるが、 iric v4 (beta版) をインストールすると sdk フォルダの下に一緒にインストールされる。

## ソルバの iRIC v4 用の修正と動作確認

ソルバを iRIC v4 用に修正する。変更内容を以下に示す。

1. (FORTRAN の場合) ソルバのソースと一緒に、SDKに含まれる iric.f90 をコンパイル・ビルドするようにする。
2. (FORTRAN の場合) ソルバのビルドの際、iRIC v3 用の cgnslib.lib, iriclib.lib の代わりに、iRIC v4 (beta版) に同梱された SDK の iriclib.lib とリンクするようにする。
3. iriclib のインターフェースの変更に対応し、ソルバ内での iriclib の関数の呼び出し処理を変更する。
4. 上記によってビルドできたら、 iRIC v4 (beta版) の solvers の下にインストールし、 iRIC v4 (beta版) 上で正常に動作するか確認する。

例えばソルバのソースコードが source.f90 だった場合、以下の処理でソルバをビルドできる。

```
ifort src\iric.f90 /MD /c
ifort src\source.f90 /MD /c
ifort *.obj lib\iriclib.lib -o solver.exe
```

* iriclib のインターフェースの変更内容については [iriclib の変更内容](iriclib_changes.md) を参照。
* 変更後の iriclib の関数を正しく呼び出せているかの確認方法は、[iriclib 関数のインターフェースの確認](iriclib_iface.md)を参照。

例として、 Nays2DH を iRIC v4 用にソースを修正した内容を以下に示す。表示されたページで "Nays2DH.f90" の中の「Load diff」をクリックする。

https://github.com/kskinoue0612/v3_nays2dh/commit/5e9e07e61e7f17a70f5ba542f88daf67707377d3#diff-f6757e8f77a14a911c7d7d652c3ed0c6318c65e668eeb9bc492e05f694e024a4

## iRIC インストーラに最新のソルバを登録する環境の作成

以下を参照。

https://github.com/i-RIC/iriclib_v4/wiki/%E3%82%BD%E3%83%AB%E3%83%90%E3%81%AE%E8%87%AA%E5%8B%95%E3%83%93%E3%83%AB%E3%83%89%E8%A8%AD%E5%AE%9A%E6%89%8B%E9%A0%86

## 備考

このページは、iRIC version 3 用のソルバの開発者の皆様に、ソルバを version 4 用に移行するための情報を提供するために一時的に作成したものです。

このページにある情報は、整理後に iRIC ソルバマニュアルに組み入れられる予定です。
