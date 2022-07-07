# iriclib の変更内容

iriclib について、 v3 と v4 の間で行われた変更内容について説明する。

iriclib については iRIC を v4 にバージョンアップするのに伴い、以下の目的でインターフェースの変更が行われた。

* cgnslib の仕様上の問題やバグが原因で、iRIC v3 は多数の計算結果を出力すると動作が遅くなったり、ハングアップして計算結果が失われたりすることがあった。これを解決するため、iriclib の内部で cgnslib を使わないようにし、また非構造格子を含む全ての入力情報を iriclib のみをつかって読み込めるようにした。その結果、安定性が向上し、計算結果の出力処理が高速になった。
* 初期に開発された関数と後から追加された関数で、統一感に問題があった。これを改善し、統一感があり分かりやすいよう名前とインターフェースを変更した。また機能の整理の中で不要になった関数を削除した。
* その他、インターフェースの明快さ向上のための変更、以前から要望があった機能追加を行った。

# 既存の関数のインターフェースの変更

既存の関数のインターフェースの変更内容について説明する。現在 iRIC v3 上で動作しているソルバを v4 用に変更するには、この章に書かれている内容に対応する。

## iriclib 関数呼び出し時の iric モジュールの利用 (FORTRAN のみ)

iRIC v3 用ソルバでは、 iriclib, cgnslib の関数を呼び出す箇所には以下の記述がある。

```
include 'cgnslib_f.h'
include 'iriclib_f.h'
```

これを削除し、代わりに以下を追加する。

```
use iric
```

これで、iric.f90 で定義された iric モジュールの関数が利用できるようになる。

## '_f' の除去 (FORTRAN のみ)

iRIC v3 用の iriclib では、 FORTRAN 用の関数は最後に '_f' がつけられていたが、iRIC v4 用の iriclib ではこれを辞め、C/C++, FORTRAN, Python で共通した名前で関数を呼び出せるようにした。

これに伴い、関数名の最後の '_f' を除去する。

変更例を以下に示す。

**変更前**
```
call cg_iric_read_integer_f('c1', c1, ier)
```

**変更後**
```
call cg_iric_read_integer('c1', c1, ier)
```

なお、本変更はエディタの機能を利用して "_f(" を "(" に一括置換することで行うことを推奨する。万が一想定外の場所まで変更してしまうことを防ぐため、変更前のファイルをバックアップした上で一括置換を行い、その後で WinMerge などを使い、想定通りの箇所が変更されていることを確認する。

## cg_iric_xxxx 関数での引数 fid の必須化

iRIC v3用の iriclib では、 cg_iric_xxxx 関数では fid は引数として不要で、複数の CGNS ファイルの入出力を行う際は、 fid が最初の引数として追加された関数 cg_iric_xxxx_mul を使う仕様になっていた。

iRIC v4用の iriclib ではこれを改め、CGNSファイルを対象に読み書きを行う関数では fid を必ず最初の引数として渡すように変更した。これは、他のファイルを取り扱うライブラリでの慣習に合わせるためである。

これに伴い、以下の変更を行う。

### cg_iric_xxxx() への引数 fid の追加

cg_iric_xxxx() を使っている箇所では、引数の先頭に fid を追加する。例を以下に示す。

**変更前**
```
call cg_iric_read_integer('c1', c1, ier)
```

**変更後**
```
call cg_iric_read_integer(fid, 'c1', c1, ier)
```

なお、ソルバの中で、 cg_open の戻り値の変数として fid 以外を利用している場合、変数名を読み替えて変更を行う。

### cg_iric_xxxx_mul() からの "_mul()" の除去

cg_iric_xxxx_mul() を使っている箇所では、単純に関数の名前から "_mul" を除去する。例を以下に示す。

**変更前**
```
call cg_iric_read_integer_mul(fid, 'c1', c1, ier)
```

**変更後**
```
call cg_iric_read_integer(fid, 'c1', c1, ier)
```

この変更は、上記で説明した '_f' の除去と同様、エディタの機能を利用して一括で行うことを推奨する。

## CGNSファイルの Open, Close 処理の変更

iRIC v4 で cgnslib を使わないようになったことを踏まえ、cg_open(_f), cg_close(_f) を使う代わりに cg_iric_open, cg_iric_close を使うように変更する。CG_MODE_MODIFY, CG_MODE_READ は IRIC_MODE_MODIFY, IRIC_MODE_READ にそれぞれ置き換える。

FORTRAN での例を以下に示す。

**変更前**
```
call cg_open(filename, CG_MODE_MODIFY, fid, ier)

...

call cg_close(fid, ier)
```

**変更後**
```
call cg_iric_open(filename, IRIC_MODE_MODIFY, fid, ier)

...

call cg_iric_close(fid, ier)
```

## cg_iric_flush の削除と cg_iric_write_sol_start, cg_iric_write_sol_end の必須化

以前は、計算結果の出力処理の最後に、 cg_iric_flush を呼び出していたが、この関数を削除した。
また cg_iric_write_sol_start, cg_iric_write_sol_end は、以前はあってもなくても動いたが、
呼び出しを必須とした。

変更前、変更後のソースの例を以下に示す。

**変更前**
```
(計算結果出力処理)

call cg_iric_flush_f(cgnsName, fid, ier)
```

**変更後**
```
call cg_iric_write_sol_start(fid, ier)

(計算結果出力処理)

call cg_iric_write_sol_end(fid, ier)
```

変更の趣旨は以下の通り。

* 以前は cg_iric_sol_write_time() で、計算結果の出力開始のための処理を同時に行っていたが、関数名と処理内容に違いがある状態になっていた。計算結果の出力開始のための処理を cg_iric_write_start() に移動し、 cg_iric_sol_write_time() では時間の出力のみ行うようにした。
* 以前は cg_iric_flush() で、計算結果の出力終了のための処理を行っていたが、関数名と処理内容に違いがある状態になっているため、名前を cg_iric_write_end() に変更した。

## ソルバとの連携機能の強化

以前は iric_check_cancel() で、ユーザが停止ボタンを押した時、ソルバを停止できるようにしていた。
これに加え、 cg_iric_check_update() を追加し、ユーザが再読み込みボタンを押した時、速やかに最新の計算結果が再読込できるようにした。

例を以下に示す。

```
imin = 0
imax = 10000
iout = 100

do i = imin, imax

  ! iRIC GUIで停止ボタンが押されたかチェック
  call iric_check_cancel(canceled)
  if (canceled == 1) then
      exit
  end if

  ! iRIC GUIで再読み込みボタンが押されたかチェック
  call cg_iric_check_update(fid, ier)

  ! ここに、計算処理の実行処理を追加

  if(mod(i, iout) == 0) then
      write(*,*) "i = ", i

      ! output
      call cg_iric_write_sol_start(fid, ier)

      ! この間に計算結果の出力処理を追加

      call cg_iric_write_sol_end(fid, ier)
  end if

end do
```

## 非構造格子の読み込み処理の変更

iRIC v3 用 iriclib では非構造格子の読み込み用機能を提供しておらず、非構造格子を読み込む際は cgnslib の関数を使用する必要があった。

iRIC v4 では cgnslib を使わないように変更したことに伴い、 iriclib の関数として非構造格子の読み込み用機能を追加した。

**変更前**

```
call cg_zone_read_f(fid, bid, zid, zonename, sizes, ier)
node_count = sizes(1)
cell_count = sizes(2)

(allocate memory here)

call cg_iric_read_grid2d_coords(fid, x, y, ier)
call cg_elements_read_f(fid, bid, zid, section, nodeids, parentdata, ier)
```

**変更後**
```
call cg_iric_read_grid_nodecount(fid, node_count, ier)
call cg_iric_read_grid_cellcount(fid, cell_count, ier)

(allocate memory here)

call cg_iric_read_grid2d_coords(fid, x, y, ier)
call cg_iric_read_grid_triangleelements(fid, nodeids, ier) ! nodeids に三角形の頂点のID のリストが格納される
```

## 関数の名前の変更

他の関数との命名規則の統一感の向上のため、名前を変更した。以下の通り、名前を変更する。

|変更前|変更後|
| ---- | ---- |
|cg_iRIC_GotoGridCoord2d|cg_iRIC_Read_Grid2d_Str_Size|
|cg_iRIC_GetGridCoord2d|cg_iRIC_Read_Grid2d_Coords|
|cg_iRIC_WriteGridCoord1d|cg_iRIC_Write_Grid1d_Coords|
|cg_iRIC_WriteGridCoord2d|cg_iRIC_Write_Grid2d_Coords|
|cg_iRIC_WriteGridCoord3d|cg_iRIC_Write_Grid3d_Coords|
|cg_iRIC_Read_Sol_GridCoord2d|cg_iRIC_Read_Sol_Grid2d_Coords|
|cg_iRIC_Read_Sol_GridCoord3d|cg_iRIC_Read_Sol_Grid3d_Coords|
|cg_iRIC_Write_Sol_GridCoord2d|cg_iRIC_Write_Sol_Grid2d_Coords|
|cg_iRIC_Write_Sol_GridCoord3d|cg_iRIC_Write_Sol_Grid3d_Coords|
|cg_iRIC_Read_Sol_Integer|cg_iRIC_Read_Sol_Node_Integer|
|cg_iRIC_Read_Sol_Real|cg_iRIC_Read_Sol_Node_Real|
|cg_iRIC_Write_Sol_Integer|cg_iRIC_Write_Sol_Node_Integer|
|cg_iRIC_Write_Sol_Real|cg_iRIC_Write_Sol_Node_Real|

## 不要になった関数の削除

不要になった関数を削除する。以下に示す関数が使われていたら削除する。

* cg_iRIC_Init
* cg_iRIC_InitRead
* cg_iRIC_GotoBase
* cg_iRIC_GotoCC
* cg_iRIC_GoToRawDataTop
* cg_iRIC_Flush
* iRIC_Check_Lock

# 新機能の追加

## 複数の計算格子での結果の出力

複数の格子での計算結果を出力する関数を追加した。
複数の格子での計算結果を出力する場合は、 名前の最後に **"_withgridid"** がついた関数を使用する。

例えば、2次元、3次元両方の結果を出力する例を以下に示す。

```
gridid_2d = 1  ! プリプロセッサで作成した2次元の計算格子の gridid は常に1

! 3次元格子を出力。 gridid_3d が戻り値で、3次元格子での計算結果を出力する際参照する。
call cg_iric_write_grid3d_coords_withgridid(fid, isize, jsize, ksize, x, y, z, gridid_3d, ier)

do while (t < tmax)
  (calculation)

  call cg_iric_write_sol_node_real_withgridid(fid, gridid_2d, 'Depth', depth, ier) ! 2次元での結果
  call cg_iric_write_sol_node_real_withgridid(fid, gridid_3d, 'Concentration', concentration, ier) ! 3次元での結果
end do
```
