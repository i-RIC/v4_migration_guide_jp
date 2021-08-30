# iriclib 関数のインターフェースの確認

iRIC v3 では、iriclib の関数の引数は iRICソルバ開発者マニュアルで確認するしかなかったが、 iRIC v4 では、SDK に含まれている iric.f90 を開くことで確認することができる。

## 単純な subroutine の例

cg_iric_read_integer の例を以下に示す。この定義から、 cg_iric_read_integer は4つの引数を持ち、先頭から順に整数、文字列、整数、整数であることが分かる。

```
  subroutine cg_iric_read_integer(fid, name, value, ier)
    integer, intent(in):: fid
    character(*), intent(in):: name
    integer, intent(out):: value
    integer, intent(out):: ier

    call cg_iric_read_integer_f2c &
      (fid, name, value, ier)

  end subroutine
```

## 引数に配列を含む subroutine の interface 例

cg_iric_read_grid_real_node の例を以下に示す。cg_iric_read_grid_real_node は interface になっていて、実際は引数の型によって "_1d", "_2d", "_3d" がついた3つの subroutine のいずれかを呼び出すことが分かる。

```
  interface cg_iric_read_grid_real_node
    module procedure cg_iric_read_grid_real_node_1d
    module procedure cg_iric_read_grid_real_node_2d
    module procedure cg_iric_read_grid_real_node_3d
  end interface
```

例えば、 cg_iric_read_grid_real_node_2d の定義を以下に示す。cg_iric_read_integer の例と同様、引数の数とそれぞれの型が定義から分かる。なお、 "_1d", "_3d" がついている subroutine は、3番目の引数である v_arr が それぞれ 1次元配列、3次元配列になっているものである。

```
  subroutine cg_iric_read_grid_real_node_2d(fid, name, v_arr, ier)
    integer, intent(in):: fid
    character(*), intent(in):: name
    double precision, dimension(:,:), intent(out):: v_arr
    integer, intent(out):: ier

    call cg_iric_read_grid_real_node_f2c &
      (fid, name, v_arr, ier)

  end subroutine
```