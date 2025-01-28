---
title: "パフォーマンス検証"
---

GPUでどれだけ高速化できるか試してみる。手元の環境はこんな感じ。

||スペック|
|--|--|
|CPU|i9-12900(Kなし) 2.4GHz|
|メモリ|CENTURY DDR4-3200 16GBx2|
|GPU|NVIDIA GeForce RTX 3060 Ti<br>4864コア GDDR6/GDDR6X 8GBメモリ<br>　MAX_THREADS_PER_BLOCK: 1024<br>　MULTIPROCESSOR_COUNT: 38<br>　WARP_SIZE: 32|
|OS|Windows11 24H2|

## Pythonの高速化アプローチ

Pythonはインタプリタ、かつ動的型付けだから遅い。なのに生成AIの共通言語になっているのは、柔軟で扱いやすく、かつ色々な高速化のアプローチができるから。らしい。
どれくらい効果あるか、4パターン試してみる。(他にもCythonとかたくさん)

1.  pure Python(3.13.1、JIT機能付き)
2.  NumPyの利用
3.  NumbaによるJIT
4.  CUDAによるGPUマルチスレッド処理

おまけで、どこでも動く代表選手`Excel VBA`と、プリミティブな`C`も比べてみる。

### Python 3.13のJIT機能

"Faster CPython"プロジェクトの一環(CPythonはPythonの公式実装)。
Pythonランタイム全体の最適化を目指したもの。デフォルトで有効、動的型付けにも対応。一部Pyston(PythonのJITコンパイラ拡張)を取り入れている。特に頻繁に実行されるコードをターゲットにして動作。有効に動作しているかどうか判別しにくい。
コンパイルされたバイナリはメモリ上に保持され、外部記憶には残らない。
> 今回はこのバージョンのCPythonをpure Pythonとします。

### NumPy

数値計算に特化した拡張モジュール。多次元配列や強力な操作関数のライブラリを提供。コア部分はCで実装。一部はCython(PythonライクなコードをC的にコンパイル)。Pythonインタフェースを実装し、PythonからCに迫るパフォーマンスの演算を叩き出せる。

### Numba

PythonコードをJITコンパイルする拡張モジュール。Python3.13のJITがPython全体の高速化を目指しているのに対し、Numbaは演算高速化に特化。
`@jit`でデコレートすると、対象関数の実行時にPythonのバイトコードを解析し、引数・戻り値の型を取得する。引数の型から関数全体の型推論を行い、LLVM(Low-Level Virtual Machine)でLLVM-IR(Intermediate Representation)を生成する。これが最終的に最適化されたネイティブコードに変換されて実行される。
コンパイルされたコードはキャッシュされ、**同一型の引数で再呼び出し**されたときに再利用される。型が異なる場合は新しいコードが生成され、関数が多重バージョン化される。
`@jit(nopython=True)`モードの場合、全くPythonのインタプリタが介在しないコードになる。Pythonが動的型付けのため、`nopython`モードで動かすのはかなり制約が厳しい。単に`@jit`だけでデコレートすると、可能な範囲で最適化してくれる。

- 可変型引数でのコールなどに弱い
- NumPyはたいがいサポートされているが一部の特殊な関数は未対応
- 型推論ができなければエラー(型ヒントを多用することになる)

通常はコンパイルした結果はメモリ上に保持し、処理が終われば破棄される。`@jit(cache=True)`で`~/.numba/`に保存されるようになる。

### CUDA

NVIDIAが提供するライブラリ。PythonからGPUを利用するコードが書ける。
CPUメモリからGPUメモリへの転送、マルチスレッド処理の実行など。
どうスレッド分割するか考えないといけないので、最初から意識して組まないとCUDAへの移行は難しい。

## 題材: 素数を数える

とりあえず素数を数えてみる。[エラトステネスの篩ﾌﾙｲ](https://ja.wikipedia.org/wiki/%E3%82%A8%E3%83%A9%E3%83%88%E3%82%B9%E3%83%86%E3%83%8D%E3%82%B9%E3%81%AE%E7%AF%A9)というアルゴリズム。

:::details 「エラトステネスの篩」の詳細

Wiki先生の説明は私には難しすぎるので、自分なりに理解を試みてみた。

1.  整数のリストを準備。ここでは上限値を20にする。
    ```
     1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20
    ```
2.  1は素数ではないので除去。
    ```
     -  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20
    ```
3.  2について、「2より大きい2の倍数」は2で割り切れるので除去。
    ```
     -  2  3  -  5  -  7  -  9  - 11  - 13  - 15  - 17  - 19  -
    ```
4.  3について、「3より大きい3の倍数」は3で割り切れるので除去。
    ```
     -  2  3  -  5  -  7  -  -  - 11  - 13  -  -  - 17  - 19  -
    ```
5.  同様に、「nについてnより大きいnの倍数」を消していく。これを繰り返して、生き残ったのが素数。シンプル。
6.  細かく考えると、nが「既に消えている数字」だったら、その倍数も消えているのでスキップしていい。この場合、4は2の倍数で消されているから、8、12、..も当然消えている。
7.  もっと細かく考えると、「nより大きい」の部分は「n * n以上の」にしていい。
    3の場合3×2＝6は除去対象だが、これは「2×3」で既に消えている。なので3×3＝9から消していけばいい。
    こうすると、5の場合「5×5＝25」から消していくが、もう20を超えている。つまりnは「上限値の平方根」まで処理すればOK。√20≒4.47なので、n＝4まで繰り返せば20までの素数リストが完成する。
:::

### パターン1: pure(純粋な)Python

なんでPythonが人気なのかよく分かりました。圧倒的にシンプル。

:::details pure Pythonのコードはこちら

```python:findprimes1_purepython.py
"""パフォーマンステスト1: 純粋なPython"""

from math import sqrt

def find_primes(limit: int):
    """
    素数を求める(pure Python)
    Args:
        limit (int): 素数を求める範囲の上限値
    Returns:
        list[int]: 上限値までに含まれる素数のリスト
    """
    # 0, 1は素数ではない。マイナス値はsqrt()がエラーになるので防止
    if limit < 2:
        return []

    # 値が全てTrueの配列を作成。範囲は0～limit
    is_prime_arr = [True] * (limit + 1)

    # 2から上限値の平方根までの範囲を確認(平方根を超える値は確認不要)
    for n in range(2, int(sqrt(limit)) + 1):

        # 素数でないと判明している値はパス
        if is_prime_arr[n]:

            # iを除くiの倍数は素数ではないので除外(スライス代入を利用)
            # pure Pythonでは通常の「スライス操作」は元配列のコピーを戻すが、
            # 「スライス代入」では**元配列が更新**される。この違いに注意
            is_prime_arr[n * n : (limit + 1) : n] = \
                [False] * len(range(n * n, limit + 1, n))

    # 包括表記で素数の配列を作成して返却
    return [n for n in range(2, limit + 1) if is_prime_arr[n]]

if __name__ == '__main__':
    print(find_primes(100))
```
配列スライスや包括表記がつよすぎる。びっくりするくらい簡単なコードになった。
素数算出のロジックは10行に満たない。Pythonがヒットした理由がよく分かる。
:::

### パターン2: NumPyの利用

更にシンプルになってしまう。恐ろしい。

:::details NumPyのコードはこちら
pure Pythonと変わらないところはコメント省略
```python:findprimes2_numpy.py
"""パフォーマンステスト2: NumPyによる高速化"""

import numpy as np

def find_primes(limit: int):
    """
    素数を求める(Numpy配列を使用)
    Args:
        limit (int): 素数を求める範囲の上限
    Returns:
        list[int]: 上限値までに含まれる素数のリスト
    """

    if limit < 2:
        # 長さ0のNumPy配列を返却
        return np.zeros(0, dtype = np.int64)

    # 値が全てTrueの配列を作成。範囲は0～limit
    # np.ones()は「全ての要素が1の配列」を生成する。
    # dtype=bool_指定で、この「1」は1バイト(8ビット)の値として生成され、
    # True値と解釈される。ちなみに全てFalseの配列はnp.zeros()で作れる。
    # 1ビットでなく1バイトなのは、そのほうが速い(ポインタで参照できる)ため。
    is_prime_arr = np.ones(limit + 1, dtype = np.bool_)

    # NumPyで可能な記法。(pure Pythonでは不可能)
    # ・NumPyにおける配列のスライスは「配列のビュー」になる
    # ・NumPyでは「右辺がスカラー値だと左辺の配列要素にブロードキャスト」される
    is_prime_arr[ : 2] = False

    for i in range(2, int(np.sqrt(limit)) + 1):
        if is_prime_arr[i]:
            # ここもブロードキャスト
            is_prime_arr[i * i : limit + 1 : i] = False

    # 値がTrueのインデックスをnp.ndarrayとして返却
    return np.nonzero(is_prime_arr)[0]

if __name__ == '__main__':
    print(find_primes(100))
```
Python配列じゃなくてNumPy配列を戻すのはちょっとフェアじゃないかもだけど、NumPyの配列操作が強すぎてpure Pythonより短くなった。
:::

### パターン3: Numba JITによる高速化

Pythonのデコレータとかいう変態機能をフル活用するNumba。

:::details Numba JITのコードはこちら
```python:findprimes3_jit.py
"""パフォーマンステスト3: Numba JITによる高速化"""

import numpy as np
from numba import jit

# @numba.jitでデコレート。これによってfind_primes()は直接実行されず、代わりに
# デコレータが呼び出される。
# Pythonでは関数もオブジェクトの一種で、デコレータはその関数への参照を受け取れる
# ため、そこからinspectモジュールなどを使って関数の構造を解析する。
# Numbaは引数から型推論を行って関数をLLVM-IRに変換し、最終的にネイティブコードを
# 生成し実行。nopython=True指定しないと「できるだけJIT」する。
# デコレータという可愛い名前で元の関数を魔改造する、恐ろしい機能。
@jit(nopython = True)
def find_primes(limit: int):
    """
    素数を求める(NumPy+Numba JIT)
    Args:
        limit (int): 素数を求める範囲の上限
    Returns:
        list[int]: 上限値までに含まれる素数のリスト
    """
    if limit < 2:
        # []と書くと「長さゼロの可変長配列」になってNumbaでは扱えない。
        # そもそもPythonが動的型付け言語だからJITと相性が悪すぎる。
        return np.zeros(0, dtype = np.int64)
    is_prime_arr = np.ones(limit + 1, dtype = np.bool_)
    is_prime_arr[ : 2] = False
    for i in range(2, int(np.sqrt(limit)) + 1):
        if is_prime_arr[i]:
            is_prime_arr[i * i : limit + 1 : i] = False
    return np.nonzero(is_prime_arr)[0]

if __name__ == '__main__':
    print(find_primes(100))
```
Numbaに`nopython=True`でJITさせるのに、型ヒントできっちり指定しないといけない。割と苦労した。ただコンパイルの必要がないからデバッグはやりやすい。
`@jit`だけでJITできるの強すぎでは？
:::

### パターン4: CUDAでGPU利用

これをやりたくてここまでやってきた。これまでとは結構違うコードになる。

:::details CUDAでGPUを使うコードはこちら
```python:findprimes4_cuda.py
"""パフォーマンステスト4: CUDAでGPU利用・並列化による高速化"""

import numpy as np
from numba import cuda

def find_primes(limit: int):
    if limit < 2:
        return np.zeros(0, dtype = np.int64)
    is_prime_arr = np.ones(limit + 1, dtype = np.bool_)
    is_prime_arr[ : 2] = False
    sqrt_limit = int(np.sqrt(limit)) + 1

    # CPUメモリ上のデータis_prime_arrをGPUメモリに転送。
    # このd_is_prime_arrはDeviceNDArray型になる
    d_is_prime_arr = cuda.to_device(is_prime_arr)

    # スレッド数・ブロック数の算出。
    # 処理データ量、GPUデバイス、計算の性質、メモリ使用量から設定。
    threads_per_block = 1024

    # cuda.grid(1)の取得結果は0～(ブロック数 * スレッド数 - 1)になるため、
    # 2～√(limit + 1)の範囲を十分カバーできるブロックサイズを求める。
    # このとき、必ず処理したい範囲を十分超えるように設定すること。
    # 不要なスレッドはスレッド内で処理を終わらせれば問題ない。
    blocks_per_grid = sqrt_limit // threads_per_block + 1

    # CUDAカーネル関数(@cuda.jitデコレートされた関数)呼び出しに特有の構文。
    # グリッドあたりのブロック数、ブロックあたりのスレッド数を指定
    # スレッド: 最小の処理実行単位。各スレッドが1つのデータ要素を処理。
    #   最大スレッド数はGPUで異なる。多くの場合1024。原則ワープ(32)の倍数。
    #   CUDAでは総スレッド数が処理対象のデータサイズに対応する。
    #   スレッド数はCUDAコア数を超えても問題ない。GPUがスレッドをスケジュー
    #   リングしてくれる。
    # ワープ: スレッド32個の集合が1ワープ。ワープ内のスレッドは同じ命令を
    #   実行する。(SIMD: Single Instruction, Multiple Data)
    # ブロック: スレッドの集合。ブロック内で各スレッドが協調して計算を実行。
    #   1～3次元の構造を持てる。画像処理・行列演算では2～3次元を利用。
    # グリッド: ブロックの集合。各グリッド内で複数ブロックが並列実行される。
    #   1～3次元の構造を持てる。グリッド数はCUDAが適切に設定してくれる。
    # この場合、並列実行される総スレッド数は
    #   blocks_per_grid * threads_per_block
    _find_primes_cuda[
        blocks_per_grid
    ,   threads_per_block
    ](
        d_is_prime_arr
    ,   limit
    ,   sqrt_limit
    )

    # GPUメモリ上のデータをCPUメモリに転送
    is_prime_arr = d_is_prime_arr.copy_to_host()
    return np.nonzero(is_prime_arr)[0]

#CUDAカーネル関数(これが一気に並列処理される本体)
@cuda.jit
def _find_primes_cuda(d_is_prime_arr, limit, sqrt_limit):

    # スレッドのグローバルインデックスを取得。
    # ブロックID * ブロック内スレッド数 + スレッドIDで生成される。
    # スレッド256, ブロック100の場合、0～25599の値になる。
    # 本来不要なスレッドも発生するので、範囲内にないインデックスの
    # 場合は単純に無視する。CUDAでは大量スレッドを走らせるほうに振って、
    # 不要なスレッドは捨てればいい。
    # Pythonのブール演算子は短絡評価だから、if()は1行のほうが速い(はず)
    i = cuda.grid(1)

    if (    2 <= i
        and i < sqrt_limit
        and d_is_prime_arr[i]):
        for j in range(i * i, limit + 1, i):
            d_is_prime_arr[j] = False

if __name__ == '__main__':
    print(find_primes(100))
```
わずか1行でGPUのマルチスレッド処理が走ってしまう。
いちばん重いのは2の倍数処理なので、これを更に分割すればもっと速くなりそう。
:::

### タイム計測用コード

:::details こんな感じ。
```python:benchmark_test.py
"""素数計算で同じ結果を得る様々なPython実装のパフォーマンスを測定"""

from contextlib import contextmanager
from datetime import datetime

from numba import cuda

import findprimes1_purepython
import findprimes2_numpy
import findprimes3_jit
import findprimes4_cuda

@contextmanager
def timer(func_name, count):
    """
    実行時間の測定
    """
    start_time = datetime.now()
    yield
    print(f'func:{func_name} count:{count}: {datetime.now() - start_time:}')

def exec_find_prime(limit, try_count):
    for i in range(1, try_count + 1):
        with timer('pure_python', i):
            findprimes1_purepython.find_primes(limit)
        # with timer('numpy', i):
        #     findprimes2_numpy.find_primes(limit)
        # with timer('numba', i):
        #     findprimes3_jit.find_primes(limit)
        # with timer('cuda', i):
        #     findprimes4_cuda.find_primes(limit)

if __name__ == '__main__':

    # GPU関連の情報
    # device = cuda.get_current_device()
    # print(f"MAX_THREADS_PER_BLOCK:  {device.MAX_THREADS_PER_BLOCK}")
    # print(f"MULTIPROCESSOR_COUNT:   {device.MULTIPROCESSOR_COUNT}")
    # print(f"WARP_SIZE:              {device.WARP_SIZE}")

    exec_find_prime(1000000000, 2)
```
:::


### ついでに1: Excel VBA

どこでも使える言語代表。変数の型指定ができるから意外に健闘するかも？

:::details Excel VBAのコードはこちら
```vb
Option Explicit ' 変数定義を強要する
Option Base 0   ' 指定ない限り配列の添字をゼロから始める

Function FindPrimes(Limit As Long) As Long()
    ' VBAでBooleanの初期値はFalseなので、ここでは「素数はFalse」とする。
    ' (配列をTrueで初期化するコストがもったいないので)
    ' 篩にかけられた値はフラグをTrueに更新していく
    Dim IsNotPrimeArr() As Boolean
    ' 結果(素数のみの配列)の格納用。この時点で数が決まらないので可変長で定義
    Dim PrimeArr() As Long
    ' 見つかった素数の数を数えるカウンタ。
    Dim PrimeCount As Long
    Dim sqrtLimit As Long
    Dim i As Long, j As Long
    ' VBAの可変長配列は長さを再定義してから使う
    ReDim IsNotPrimeArr(Limit)

    If Limit < 2 Then
        ' VBAでは長さ未定義の配列への参照がエラーになるため、
        ' 添字0～0の配列を作成して戻す
        ReDim PrimeArr(0)
        ' VBAでは関数名への代入がReturn相当になる
        FindPrimes = PrimeArr
        Exit Function
    End If

    sqrtLimit = CLng(Sqr(Limit)) + 1
    ' VBAでは「値がFalseのみの配列」を一発で生成できないので、
    ' 素数を求めながら素数の数を数える(篩に掛けられた数を減算)
    ' 最初に-1しているのは配列が0から始まるため。
    PrimeCount = Limit - 1

    For i = 2 To sqrtLimit
        ' 分かりにくいが「まだ素数でないと判定されていない場合」
        If Not IsNotPrimeArr(i) Then
            For j = i * i To Limit Step i
                ' 素数を数えるため、無条件にフラグを更新できない
                If Not IsNotPrimeArr(j) Then
                    ' ここまできたら「素数でない」と確定
                    IsNotPrimeArr(j) = True
                    ' 素数の数を減算
                    PrimeCount = PrimeCount - 1
                End If
            Next j
        End If
    Next i

    ' 最終的に残った数＝素数の数で配列を作成。0スタートなので-1
    ReDim PrimeArr(PrimeCount - 1)
    j = 0
    For i = 2 To Limit
        If Not IsNotPrimeArr(i) Then
            ' 素数のみ配列に格納
            PrimeArr(j) = i
            j = j + 1
        End If
    Next i
    FindPrimes = PrimeArr
End Function

Sub FindPrimesVBA()
    Dim t As Single
    Dim PrimeArr() As Long
    Dim i As Long
    Dim s As Single, e As Single
    For i = 0 To 10
        s = Timer
        PrimeArr = FindPrimes(1000000000)
        e = Timer
        Debug.Print CStr(i) & ": " & CStr(e - s)
    Next i
End Sub
```
Pythonほど自由な配列操作ができないので長くなった。Scripting.Dictionaryを使えばもっとシンプルに書けるけど大概遅くなるし、番外なのでこれくらいで。
:::

### ついでに2: C

ン十年ぶりなのでGPT先生に手伝ってもらった。

:::details Cのコードはこちら
便利な配列処理がないからVBAに似た感じになる。更にメモリを直接扱うからややこしい。
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <math.h>
#include <time.h>

// 戻り値は素数のint配列の先頭へのポインタ、prime_countは見つけた素数の数へのポインタ
// ※ポインタ＝メモリアドレス。その参照先は*prime_countで表現する。
int* FindPrime(int limit, int *prime_count) {

    if (limit < 2) {
        *prime_count = 0;
        return NULL; // 素数なし
    }

    // Cでは配列に必要な量のメモリを自分で確保しないといけない。
    // 処理系によるが自分の環境ではsizeof(bool)は1(バイト)
    // calloc()は領域を0クリアする→falseと評価される
    bool *is_not_prime = (bool *)calloc(limit + 1, sizeof(bool));

    int sqrt_limit = (int)sqrt(limit) + 1;
    //VBAと同じく「素数でない数」を減算して求める。
    *prime_count = limit - 1;

    for (int i = 2; i <= sqrt_limit; i++) {
        if (!is_not_prime[i]) {
            for (int j = i * i; j <= limit; j += i) {
                if (!is_not_prime[j]) {
                    is_not_prime[j] = 1; // 素数から除外
                    (*prime_count)--; // 素数を減算
                }
            }
        }
    }

    // 素数配列を作成(これは初期化不要のためmalloc())
    int *primes = (int *)malloc((*prime_count) * sizeof(int));

    int j = 0;
    for (int i = 2; i <= limit; i++) {
        if (!is_not_prime[i]) {
            primes[j++] = i;
        }
    }

    free(is_not_prime); // 使用済みメモリを解放
    return primes;
}

int main() {
    const int PRIMES_LIMIT = 1000000000;

    for(int i = 0; i<=10; i++) {
        clock_t t = clock();
        int prime_count = 0;
        int *primes = FindPrime(PRIMES_LIMIT, &prime_count);
        clock_t e = clock();
/*
        printf("\n");
        for (int i = 0; i < prime_count; i++) {
            printf("%d ", primes[i]);
        }
*/
        free(primes); // 使用済みメモリを解放
        printf("time: %.4f\n", (double)(e - t) / CLOCKS_PER_SEC);
    }
    return 0;
}
```
Cは配列宣言の代わりに「配列に必要なサイズのメモリを確保」。使い終わったメモリは手動で開放。他言語でこれが不要なのはガベージコレクタ※機構のおかげ。また、Cでは配列の上限チェックとかないから**範囲を超えて関係ないデータを蹂躙できる**。Javaが登場したとき、「閾値を超える配列アクセスはエラーになるから安全だよ」という仕様が、C/C++過激派から「無駄に重いだけ」と批判された。Cが速いのはこういうところのトレードオフ。
> メモリ上にデータがあって、そのデータに対する参照がゼロになったとき、そのデータは「誰からも必要とされていない」ので片付けるのが(広義の)ガベージコレクタ(狭義では「循環参照の片付け」を指す)。この仕組みで一般にパフォーマンスは悪くなる。特にPythonはスカラー型が悉くimmutableだから、このコストは馬鹿にならない(気がする)。
:::

- 各プログラムで1,000,000,000(10億)までの素数を求める。
- なるべく条件を揃えるため、毎回Windowsをサインアウトしてから入り直し、不要なプロセスできるだけ落とし、CPUが落ち着くのを待ってからコンテナ上げて計測。Cも含めて全てコンテナ内で実行(VBA以外)。
- 初回はJITが不利になるから、同じ処理を11回続けて実行し、2～11回の平均で比較。

## 結果

![](/images/tr-windows-ve/exec_time_result.png)

- purePythonはVBAに負けるかも？と思ったけど、辛うじて勝利。
- NumPyが一番速い。Cより速い
- NumbaでJITしたら僅かに遅くなる。10回全部負けてるから誤差とも思えない。
- そして**CUDAが惨敗**。

各回の計測結果はこんな感じ。(秒)

|#|purePython|NumPy|Numba|CUDA|Excel VBA|C|
|--|--|--|--|--|--|--|
|0|~~43.189~~|~~7.171~~|~~7.635~~|~~15.579~~|~~39.910~~|~~9.402~~|
|1|39.756|7.026|7.318|14.523|41.324|9.177|
|2|39.563|7.082|7.202|14.475|40.680|9.000|
|3|39.630|7.026|7.301|14.448|40.523|8.983|
|4|38.882|6.978|7.270|14.488|42.629|8.990|
|5|39.399|6.880|7.187|14.465|40.758|9.257|
|6|39.188|6.881|7.278|14.485|41.734|9.049|
|7|40.119|7.092|7.261|14.521|41.305|9.195|
|8|39.136|7.050|7.176|14.523|41.691|9.208|
|9|39.235|6.877|7.274|14.445|41.328|9.381|
|10|39.375|6.910|7.179|14.466|42.133|9.222|
|**1～10平均**|**39.428**|**6.980**|**7.245**|**14.484**|**41.411**|**9.146**|

気になるところをもうちょい細かく。

### NumPy VS C

![](/images/tr-windows-ve/numpy-vs-c.png)

|処理|NumPy|C|
|--|--|--|
|配列初期化|0.089|0.000|
|素数計算|6.209|8.242|
|素数のみの配列作成|0.836|1.232|

図り直したから数字変わってるけど、NumPyは条件によっては並列処理をするらしく、その差が出てる感じ。

### CUDAでGPU

問題がこれ。

![](/images/tr-windows-ve/cuda-performance.png)

はい分かりました。**素数計算は0.0003秒で終わってます**。

|処理|時間(sec)|
|--|--|
|全ての値がTrueの配列作成|0.09640|
|GPUメモリに転送|0.11164|
|素数計算|0.00027|
|結果をCPUメモリに転送|13.5983|
|素数のみの配列作成|0.67821|





## おまけ: Pythonと生成AIの歴史

Pythonはインタプリタだし動的型付け言語のため、パフォーマンスはCやC++とは比較にならないほど悪い。
それでもLLM等の生成AI分野で共通言語になっているのは、NumPy, SciPy, Pandas, TensorFlow, PyTorchのような強力なライブラリを持ち、更にCUDAによるGPUのマルチスレッドプログラミングも容易だから。
C++で書けばと思うが、プログラマでない研究者にC++は硬派がすぎる。生成AI研究は膨大な試行錯誤が必要で、サッと書いてサッと実行でき、直感的に分かりやすいPythonが支持されてきた。

(GPT先生に教えてもらった歴史経緯)

1.  初期のAI研究とPythonの選択(1990年代後半〜2000年代初頭)
    - 1991年Pythonリリース。この時期は主に機械学習(ML)、ルールベースAIが中心。
    - シンプルでわかりやすい構文が研究者やエンジニアから評価。
    - 2006年NumPyリリース、SciPyなどの科学計算ライブラリが徐々に普及。Pythonがプラットフォームとして機能し始める。

2.  深層学習とPythonの台頭(2010年代初頭)
    - 生成AIの基盤技術(特にニューラルネットワーク)が躍進。
    - 2012年AlexNetがImageNetコンペティションで優勝、深層学習(Deep Learning)が注目。
    - Pythonライブラリの台頭:
      TensorFlow(2015年)、PyTorch(2016年)リリース。フロントエンドPython、バックエンドC++で高速・容易に試行錯誤できる環境がAIモデル開発を加速。

3.  生成AIの登場(2014年〜2017年: トランスフォーマーの誕生)
    - 2014年、GAN(生成的敵対ネットワーク)発表。生成AIの可能性が広まる。
    - 2017年、Googleがトランスフォーマー(Transformer)モデルを発表(Attention Is All You Need)。自然言語処理(NLP)や生成AIが次の段階へ。
      トランスフォーマーモデル(BERT、GPTなど)はPython主軸になる。Hugging Face(2018年設立)がトランスフォーマーモデルをPythonベースで簡単に利用できるライブラリを提供。

4.  生成AIの実用化とPythonの標準化(2018年〜)
    - OpenAI、DeepMind、Anthropicなど大規模生成AIモデル開発、PythonのAPIが普及。クラウドプラットフォームがPython向けSDKを提供。
    - 生成AIのツールやフレームワークがPythonをベースに標準化。プロトタイプから商用環境までPythonで通せることが評価。
