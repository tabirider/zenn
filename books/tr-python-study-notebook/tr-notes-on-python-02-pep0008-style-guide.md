---
title: "PEP 8 – Style Guide for Python Code(コードのスタイルガイド)"
---

[和訳](https://pep8-ja.readthedocs.io/ja/latest/#id3)より。短くしただけ、正確には本家参照。※は補記
Post-History: 05-Jul-2001, 01-Aug-2013

> PEPはPythonに関する新機能や改善案の文書。Pythonコミュニティで採択されると公式になる。PEP8はコードスタイルのガイドライン。標準ライブラリには厳格に適用されている。

## 一貫性にこだわりすぎるのは狭い心の現れ

  可読性重要。ただし**一貫性を崩すべき場合**も。

  - ガイドに従うとコードが読みにくくなる場合
  - ガイドに沿わない周囲のコードに合わせる(なんなら逆に掃除しちまえ)
  - ガイドより前に書かれたもの
  - ガイド推奨機能サポート前バージョンとの互換性

## コードレイアウト

### インデント

- スペース4つ
- 行継続は折り返された要素を揃える
- `()` `[]` `{}`は暗黙の行結合を利用
- 突き出しインデントは最初の行に引数を書かず継続行をインデント

  ```python
  # function
  # 開き括弧に揃える
  foo = long_function_name(var_one, var_two,
                          var_three, var_four)
  # 引数とそれ以外を区別するため、スペースを4つ(インデントをさらに)加える
  def long_function_name(
          var_one, var_two, var_three,
          var_four):
      print(var_one)
  # 突き出しインデントはインデントのレベルを深くする
  foo = long_function_name(
      var_one, var_two,
      var_three, var_four)
  ```

- 複数行継続時はスペース4でなくてもいい

  ```python
  # 突き出しインデントの場合は、スペース4つに限らない
  foo = long_function_name(
    var_one, var_two,
    var_three, var_four)
  ```

- `if`の複数行はスペース4で揃える、`if (`で4文字。けどネスト内部と見分けがつかなくなるから他も許容

  ```python
  # 追加インデントなし
  if (this_is_one_thing and
      that_is_another_thing):
      do_something()

  # コメントはエディタがシンタックスハイライトするから区切りに使える
  if (this_is_one_thing and
      that_is_another_thing):
      # 両方の条件がtrueなので、処理を調整可能
      do_something()

  # 継続された行の条件をインデントする
  if (this_is_one_thing
          and that_is_another_thing):
      do_something()
  ```

- 複数行カッコ閉じは2種類

  ```python
  # カッコはどちらでも
  my_list = [
      1, 2, 3,
      4, 5, 6,
      ]
  my_list = [
      1, 2, 3,
      4, 5, 6,
  ]
  ```

### タブではなくスペース

### 1行は79文字

- できるなら72文字までにしろ。長いとエディタが折り返して見にくい。
- 責任を持つチームの合意で99文字まで緩和してもいい。
  その場合でも`docstring`やコメント等は72文字まで。
- `\`よりカッコで囲んで暗黙行継続のほうがいい。
- 3.10までは`With`が暗黙の行継続できない。

  ```python:※before Python3.10
  with open('/path/to/some/file/you/want/to/read') as file_1, \
      open('/path/to/some/file/being/written', 'w') as file_2:
      file_2.write(file_1.read())
  ```

- `assert`の行継続は`\`のほうがいい。

  ```python:※assertの行継続
  assert  condition1 \
      and condition2 \
      and condition3 \
      and condition4
  ```

- 行継続のときはきちんとインデントしろ。

### 2項演算子と改行

どっちでもいいが演算子前の改行を推奨。

```python
income = (gross_wages
          + taxable_interest
          + (dividends - qualified_dividends)
          - ira_deduction
          - student_loan_interest)
```

### 空行

- トップレベルの関数・クラスは2行空ける
- クラス内メソッドは1行。2行空けてもいいが控えめに
- ワンライナー(ダミー実装等)は空行省略してもいい

  ```python:※ワンライナー
  class Hoge:

      def __init__(self):
          self.x = 0

      def piyo1(self): pass # Dummy
      def piyo2(self): pass # Dummy
      def _addx(self, x): self.x += x # Utility
  ```

- 関数内の空行はロジックの境目を示す程度、控えめに
- Pythonは`^L`(ページ区切り)を空白と認めている、ファイル内の分割に使ってよい
  (Webビューアとかでも機能すると期待するな)

### エンコーディング

**常にUTF-8**で書け、エンコ宣言するな。
ASCII以外の文字は控えめに、英語覚えろ。オープンソースはだいたいそうなってる。

### import

- 通常は行を分ける。

  ```python
  # これは良い
  import os
  import sys
  # これはダメ
  import sys, os
  ```

- モジュール内オブジェクトはインラインでOK

  ```python
  # これは良い
  from subprocess import Popen, PIPE
  ```

- 常にファイルの先頭。モジュールコメント・docstringの後、グローバル変数の前。
- 順番は以下の通り。**1行の空白で区切る**。
  1. 標準ライブラリ
  2. サードパーティ関連
  3. ローカルアプリ・ライブラリ固有
- 絶対インポートを推奨。標準ライブラリは常に絶対パス。

  ```python
  # 絶対インポートで
  import mypkg.sibling
  from mypkg import sibling
  from mypkg.sibling import example
  ```

- 冗長なときは相対でも。複雑な(※≒階層の深い)パッケージとか。

  ```python
  # 冗長を避けるための相対インポート
  from . import sibling
  from .sibling import example
  ```

- クラスインポートはこの形でもOK

  ```python
  # クラスだけインポートはこれでも良い。
  from myclass import MyClass
  from foo.bar.yourclass import YourClass
  # 衝突が起きたら明示インポートで回避
  import myclass
  import foo.bar.yourclass
  ```

- ワイルドカード(`from modulr import *`)やめろ、可読性低下。
  内部インタフェースをAPIとして公開する場合は許す。

  ```python
  # (API再公開のようなケースを除き)不可
  from <module> import *
  ```

### モジュールレベルの \_\_変数名\_\_

モジュール`docstring`の後、`from __future__`以外の全ての`import`より前。
```python
"""This is the example module.

This module does stuff.
"""

from __future__ import barry_as_FLUFL

__all__ = ['a', 'b', 'c']
__version__ = '0.1'
__author__ = 'Cardinal Biggles'

import os
import sys
```

## 文字列に含まれる引用符

`'string'`と`"string"`は同じ、どっちでもいいけど決めておけ。
`'`や`"`を文字列に含む場合、`\`でエスケープするよりもう一方の引用符を使うのがいい。

## 式や文中の空白

### 余計な空白禁止

- カッコ直後、閉じる直前

  ```python
  # 正しい:
  spam(ham[1], {eggs: 2})
  # 間違い:
  spam( ham[ 1 ], { eggs: 2 } )
  ```

- カンマと閉じカッコの間

  ```python
  # 正しい:
  foo = (0,)
  # 間違い:
  bar = (0, )
  ```

- `,`, `;`, `:`の直前

  ```python
  # 正しい:
  if x == 4: print(x, y); x, y = y, x
  # 間違い:
  if x == 4 : print(x , y) ; x , y = y , x
  ```

- スライスは`:`が2項演算子的に振る舞う。`:`は優先度最低の演算子と扱われるから、左右均等に空白を挟んでよい。
- 例外: スライスのパラメータ省略時はスペースも省略

  ```python
  # 正しい:
  ham[1:9], ham[1:9:3], ham[:9:3], ham[1::3], ham[1:9:]
  ham[lower:upper], ham[lower:upper:], ham[lower::step]
  ham[lower+offset : upper+offset]
  ham[: upper_fn(x) : step_fn(x)], ham[:: step_fn(x)]
  ham[lower + offset : upper + offset]
  # 間違い:
  ham[lower + offset:upper + offset]
  ham[1: 9], ham[1 :9], ham[1:9 :3]
  ham[lower : : step]
  ham[ : upper]
  ```

- 関数呼び出しの引数前カッコ

  ```python
  # 正しい:
  spam(1)
  # 間違い:
  spam (1)
  ```

- インデックス・スライス開始カッコの直前

  ```python
  # 正しい:
  dct['key'] = lst[index]
  # 間違い:
  dct ['key'] = lst [index]
  ```

- 代入(や他の)演算子を揃えるための、演算子周囲の1つ以上のスペース

  ```python
  # 正しい:
  x = 1
  y = 2
  long_variable = 3
  # 間違い:
  x             = 1
  y             = 2
  long_variable = 3
  ```

### その他の推奨事項

- 行末の空白残さない
  - `\`後のスペースは行継続と見なされない
- 2項演算子の両側は1つだけスペース
  - 代入演算子`=`
  - 拡張代入演算子`+=`, `-=`など
  - 比較演算子`==`, `<`, `>`, `!=`, `<>`, `<=`, `>=`, `in`, `not in`, `is`, `is not`
  - ブール演算子`and`, `or`, `not`
- 優先順位が違う演算子の場合、一番低い演算子の両側に同じ数のスペース入れてもよい。

  2つ以上は絶対使うな
  ```python
  # 正しい:
  i = i + 1
  submitted += 1
  x = x*2 - 1
  hypot2 = x*x + y*y
  c = (a+b) * (a-b)
  # 間違い:
  i=i+1
  submitted +=1
  x = x * 2 - 1
  hypot2 = x * x + y * y
  c = (a + b) * (a - b)
  ```

- 関数アノテーションは`:`のルール(前にスペース入れない)に加え`->`前後に常にスペース。

  ```python
  # 正しい:
  def munge(input: AnyStr): ...
  def munge() -> PosInt: ...
  # 間違い:
  def munge(input:AnyStr): ...
  def munge()->PosInt: ...
  ```

- デフォルト値持ちの引数アノテーションと組み合わせるときは`=`の前後にスペース

  ```python
  # 正しい: (arg: Type = val) (arg=val)
  def munge(sep: AnyStr = None): ...
  def munge(input: AnyStr, sep: AnyStr = None, limit=1000): ...
  # 間違い:
  def munge(input: AnyStr=None): ...
  def munge(input: AnyStr, limit = 1000): ...
  ```

- 複合文(1行に複数文)は基本ダメ。

  ```python
  # 正しい:
  if foo == 'blah':
      do_blah_thing()
  do_one()
  do_two()
  do_three()
  # 間違い:
  if foo == 'blah': do_blah_thing()
  do_one(); do_two(); do_three()
  ```

- `if`/`for`/`while`に短い文を1行はOKなこともあるが、複数行で構成されるステートメントでの複合文は絶対ダメ。当然、長くなった複合文の折り返しも絶対ダメ。

  ```python
  # あまり良くない
  if foo == 'blah': do_blah_thing()

  for x in lst: total += x

  while t < 10: t = delay()

  # 絶対ダメ
  if foo == 'blah': do_blah_thing()
  else: do_non_blah_thing()

  try: something()
  finally: cleanup()

  do_one(); do_two(); do_three(long, argument,
                              list, like, this)

  if foo == 'blah': one(); two(); three()
  ```

## 末尾カンマ

通常は任意。要素が1つのタプルの場合、例外的に必須。技術的に冗長でもカッコ閉じを推奨。

```python
# 正しい:
FILES = ('setup.cfg',)
# 間違い:
FILES = 'setup.cfg',
```

```python:※注釈
print(type((1)))  #class 'int'
print(type((1,))) #class 'touple'
```

末尾カンマは冗長でも便利な場合がある。値、引数、`import`された値のリストが後で伸びそうなとき、バージョン管理システム使ってるとき。末尾`,`で次行カッコ閉じれば行追加が容易。`,)`みたいな記法は意味がない(上の要素数1のタプルを除く)。

```python
# 正しい:
FILES = [
    'setup.cfg',
    'tox.ini',
    # ※注釈 後で追加しても上の行を変えずに済む→diffしやすい
    ]
initialize(FILES,
           error=True,
           )
# 間違い:
FILES = ['setup.cfg', 'tox.ini',]  # ※注釈 diffし易さ目的だからこれは無意味
initialize(FILES, error=True,)
```

## コメント

**コメントの更新は最優先**
複数の完全な文で書くべき。`The first word`大文字から。
ブロックコメントは一つ以上の段落、段落は複数の完全な文。`.`で終わる。最後の文を除き`.`の後にスペース。英語圏外の人は、自分と違う言葉の人に120%読まれない限り英語で書け。

### ブロックコメント
その後に続くコードに適用(コードの前に書け)、コードと同じレベルにインデント。
各行は`#`の後スペースで始める。ブロックコメント内の段落は`#`だけの1行で区切る。

### インラインコメント
**控えめに**。文とインラインコメントの間には2個以上のスペース。`#`の後にスペース。自明なことにコメント入れるな邪魔、意味があるときだけ書け。

```python
# ※意味のないインラインコメント
x = x + 1                 # Increment x
# ※意味のあるインラインコメント
x = x + 1                 # 境目を補う
```

### docstrings

[PEP 257 – Docstring Conventions](https://peps.python.org/pep-0257/)参照
全ての公開モジュール・関数・クラス・メソッドにdocstring。非公開メソッドには不要だが、何をしているのかは説明すべき。複数行のdocstringは`"""`だけの行で閉じる。1行のdocstringは`"""comment"""`していい。

```python
"""Return a foobang

Optional plotz says to frobnicate the bizbaz first.
"""
```

## 命名規約

完全な一貫性を持たせるつもりはない(ライブラリのほうが混乱してる)。これから書くときは現在の推奨に従え。既存ライブラリが違ってたら内部に一貫性を持たせるのが良い。

### いちばん重要な原則

公開APIは実装より使い方の名前にしろ。

### 実践されている命名方法

`b`, `B`, `lowercase`, `snake_case`, `UPPERCASE`, `SCREAMING_SNAKE_CASE`, `CapWords`, `camelCase`

:::details ※注釈 ざっくり他言語と比べてこんな感じ。なおPythonはケースセンシティブ

|識別子|C#|Python|
|--|--|--|
|パッケージ（名前空間）|`CapWords`|`lowercase`(※アンスコ非推奨)|
|モジュール|`CapWords`|`snake_case`|
|クラス|`CapWords`|`CapWords`|
|型変数|`CapWords`|`CapWords`|
|例外|`CapWords`|`CapWords`|
|グローバル変数(Pythonにモジュール超えは無い)|`CapWords`|`snake_case`|
|パラメータ（引数）|`camelCase`|`snake_case`|
|メソッド（関数）|`CapWords`|`snake_case`|
|変数|`camelCase`|`snake_case`|
|定数(Pythonには無い)|`CapWords`|`SCREAMING_SNAKE_CASE`|

:::


Pythonはクラス名・モジュール名で名前空間切ってんだから`X11`みたいなprefix付けるな。`os.stat()`は`st_mode`とか戻すけどPOSIXに準じてるだけだから気にするな。

アンスコ
- `_hoge`: 内部で使う意味。`from M import *`でもインポートしない。
- `hoge_`: Pythonキーワードとの衝突を防ぐ。
- `__hoge`: **マングリング発動**
- `__hogehoge__`: **再発明するな**ドキュメントにあるのだけ使え

### 守るべき命名規約

#### l, O, Iだけの変数

**絶対使うな**見分けつかん。

#### ASCIIとの互換性

標準ライブラリの識別子はASCII互換。
[PEP 3131 – Supporting Non-ASCII Identifiers](https://peps.python.org/pep-3131/)の[policy section](https://peps.python.org/pep-3131/#policy-specification)参照。

#### パッケージ・モジュール

- モジュールは全て小文字の短い名前。アンスコ使ってもいい。
- パッケージは小文字の短い名前、アンスコ非推奨。
- C/C++の拡張モジュールに高レベルのPythonインタフェースが付いてるときは、C/C++モジュールはアンスコ付ける(`_socket`)。

#### クラス

常に`CapWords`。callable※、インタフェースとしてドキュメント化されてるのは関数向け規約でもいい。


:::details ※callable: `__call__`を実装したクラス。関数的に呼び出せる。
```python
class some_class:  # クラスだけど関数的に使えるから関数的な名前
    def __call__(self):
        print('hoge')

some_obj = some_class()
some_obj()  # hoge
```

デコレータとして使える。

```python
class some_decorator:
    def __call__(self, func):
        def wrapper(*args, **kwargs):
            print('hoge')
            res = func(*args, **kwargs)
            print('piyo')
            return res
        return wrapper

@some_decorator()
def some_func():
    print('fuga')

some_func()  #hoge -> piyo -> fuga
```
:::

ビルドインクラスはほぼ単一の単語だが例外もある。

#### 型変数

[PEP 484 – Type Hints](https://peps.python.org/pep-0484/)で導入された型変数※は通常`CapsWords`方式。短い名前の方が良い: `T`, `AnyStr`, `Num`。
:::details ※型変数(Type Variables): 「型は決めないけど引数と戻り値は同じ型だよ」みたいな表明の仕組み。

```python
from typing import TypeVar, List

T = TypeVar('T')  # 型変数Tを定義

def repeat(value: T, count: int) -> List[T]:
    return [value] * count
```
これで「引数valと戻り値の配列に含まれる値は同じだよ」と表明している。
型ヒントに反していてもPython自体はエラーを出さないが、`mypy`はエラー検出してくれる。
:::

共変※や反変※の変数は`_co`や`_contra`のサフィックスを推奨。

```python
from typing import TypeVar

VT_co = TypeVar('VT_co', covariant=True)
KT_contra = TypeVar('KT_contra', contravariant=True)
```

:::details 共変(covariant): サブタイプが許容される。
```python
from typing import TypeVar
T_co = TypeVar('T_co', covariant=True)  # 共変な型変数
```
この型はサブクラス(子クラス、派生クラス)も許容する。
:::

:::details 反変(contravariant): スーパータイプが許容される。
```python
from typing import TypeVar
T_contra = TypeVar('T_contra', contravariant = True)  # 反変な型変数
```
この型はスーパークラス(親クラス)も許容する。
:::

#### 例外

クラスにすべき。クラス命名規約に従え。エラーは`Error`を付ける。

#### グローバル変数

（※モジュールで名前空間が分離されるため、他言語のような「グローバル」変数はない）
`from M import *`前提のモジュールは`__all__`で不要なの隠すか、見せたくない変数を`_val`しろ。関数レベルでも同じ。(※`from M import *`自体が非推奨)

#### 関数・変数

小文字、アンスコで区切るべき。`snake_case`
> `camelCase`は後方互換のみ許可。

#### 引数

- インスタンスメソッドの第1引数は常に`self`
- クラスメソッドの第1引数は常に`cls`
- 予約語と衝突するとき、`class_`のほうが`clss`よりマシ。というか予約語使うな

#### メソッド名・インスタンス変数

- 関数の規約に従う。
- サブクラスとの衝突はマングリングで回避。

#### 定数

モジュールレベルで定義。`SCREAMING_SNAKE_CASE`。
※Pythonに定数はないので、それっぽく見せるだけ。

#### 継承の設計

- 公開・非公開・サブクラスだけ公開をちゃんと考えろ(※ざっくり意訳)
  - public: `func_name`
  - protected: `_func_name`
  - private: `__func_name`(マングリング(衝突しないとは言ってない))
    - マングリングは`__getattr__()`とかデバッグで面倒だけど規則性ある、文句言うな
    - マングリング嫌いな人もいるかもしれんバランス考えろ
- 分からないなら公開するな、後で隠すのが大変
- 使う側は`public`に後方互換性を期待する(公開したら下手にいじるな)。
  `_non_public`は消されても文句言うな
- `private`って言うな、Pythonにそんなんない
- とりあえずプロパティだけ公開しろ、ややこしいアクセサ(※`getter`)やミューテータ(訳注: 内部状態を変更するメソッド(※(`setter`))書くな。必要なら後で関数化できる(※`@property`とか`@hoge.setter`)
  - 関数的なのは副作用に気をつけろ、キャッシュはいい
  - プロパティで重い処理するな、使う側は想定してない

### 公開インタフェースと内部インタフェース

- 後方互換は公開IFだけ考えればいい。
- ドキュメント化されてるのは(違うと書いてない限り)公開IF。
- イントロスペクションのため公開属性を`__all__`で宣言すべき。`__all__`空にすれば非公開の意味になる。(※`__all__`に入れてなくても`import`できる。目印以上の意味はない)
- `__all__`しても`_func()`すべき。
- `import`された名前は実装の詳細を表現しているとみなすべき。

## プログラミングの推奨事項

- `CPython`以外の実装(`PyPy`, `Jython`, `IronPython`, `Cython`, `Psyco`等)に不利にならないコードを書け。`a += b`はリファレンスカウントが入っていないPythonにはない。パフォーマンスに敏感なところは`''.join()`を使え、文字列結合が線形時間で終わる。

- `None`のようなシングルトンとの比較は必ず`is`, `is not`を使え、等値演算子を使うな。
  `if x is not None`のつもりで`if x`を書くと、`None`じゃなくても`False`になったりするから気をつけろ(※ダメとは言ってない)。
  `not ... is ...`じゃなくて`is not`を使え。読みやすいからな。

  ```python
  # 正しい:
  if foo is not None:
  # 間違い:
  if not foo is None:
  ```
- 拡張比較(rich comparison)でソート実装するときは`__eq__`, `__ne__`, `__lt__`, `__le__`, `__gt__`, `__ge__`全部書くのがベスト。`functools.total_ordering()`デコレータが比較メソッド付けてくれる。[PEP207 - Rich Comparisons](https://peps.python.org/pep-0207/)は反射律(`y > x`なら`x <= y`)を想定してるけど、`sort()`と`min()`は`<`を使うし`max()`は`>`を使う。

- `f = lambda`じゃなくて`def f`しろ、関数名tracebackできん。`lambda`は他の式に埋める他に利点がない。(※引数に関数を渡すのがある。`map()`とか)

- `Exception`を継承しろ、`BaseException`はほとんどキャッチしなくていいときだけ。
  例外の階層は「例外を投げる人」じゃなくて「キャッチする人」を考えろ、何が起こったか分かるようにしとけ(ビルトイン例外で混乱した教訓が[PEP 3151 – Reworking the OS and IO exception](https://peps.python.org/pep-3151/)にあるから読んどけ※)。

  > ※PEP3151: OS/IO関連例外の再構築(3.3～)。`FileNotFoundError`とかなかったから、`OSError`を捕まえて`e.errno != errno.ENOENT`とかしてたのを修正。

  クラスの命名規約に従う、エラーなら`Error`から始めろ。他のフロー制御やシグナル目的なら付けないでいい。

- tracebackを失わないよう、例外チェインは`raise X from Y`しろ。内部の例外を入れ替える(`raise X from None`)なら元の詳細を新しい例外に明示しろ。`KeyError`を`AttributeError`にするなら、`KeyError`の属性を保護したり、元のメッセージを新しい例外に埋めるとか。

- `except:`は`except BaseException:`と同義。キャッチする例外をちゃんと書け、`Ctrl`+`C`できないし他の例外をもみ消すぞ。シグナル的なのもキャッチするときは`except Exception:`しろ。
  ```python
  try:
      import platform_specific_module
  except ImportError:
      platform_specific_module = None
  ```
  生`exception:`の使い途はだいたい2つ。
  1. 例外ハンドラが`traceback`するかログ取るとき(ユーザがエラーを認識できる)
  2. リソース後始末の後、上流に`raise`するとき(`try`..`finally`のほうがいいかもよ)

- OSエラーは(3.3以降)`errno`を見るより必要な例外種類を`except`しろ。

- 本当に必要なとこだけ`try`しろ。バグをもみ消すな。

  ```python
  # 正しい:
  try:
      value = collection[key]
  except KeyError:
      return key_not_found(key)
  else:
      return handle_value(value)
  # 間違い:
  try:
      # try で囲む処理が大きすぎる！
      return handle_value(collection[key])
  except KeyError:
      # handle_value() が発生させる KeyError もキャッチする
      return key_not_found(key)
  ```

- リソースを特定部分だけで使うときは`with`しろ、確実に後始末できる。`try`/`finally`でもいい。

- コンテキストマネージャをリソース取得・開放以外に使うなら、別の関数・メソッド名で`with`すべき。(※`__del__()`は確実に直後に呼ばれると保証されてない)
  ```python
  # 正しい:
  with conn.begin_transaction():
      do_stuff_in_transaction(conn)
  # 間違い: __enter__, __exit__の動きが分からん
  with conn:
      do_stuff_in_transaction(conn)
  ```

- `return`は書き方揃えろ。確実に`return`するか完全に返さないか。時々`return`しないより`return None`書け。(できるなら)最後1つがいい。

  ```python
  # 正しい:
  def foo(x):
      if x >= 0:
          return math.sqrt(x)
      else:
          return None
  def bar(x):
      if x < 0:
          return None
      return math.sqrt(x)

  # 間違い:
  def foo(x):
      if x >= 0:
          return math.sqrt(x)
  def bar(x):
      if x < 0:
          return
      return math.sqrt(x)
  ```

- プレフィクス・サフィックスのチェックは文字列スライスじゃなくて`''.startswith()`と`''.endswith()`を使え、分かりやすい。

  ```python
  # 正しい:
  if foo.startswith('bar'):
  # 間違い:
  if foo[:3] == 'bar':
  ```

- オブジェクト型の比較は型名を比べるな、`isinstance()`を使え。

  ```python
  # 正しい:
  if isinstance(obj, int):
  # 間違い:
  if type(obj) is type(1):
  ```

- シーケンス(文字列、リスト、タプル)は空シーケンスが`False`なことを使っていい。

  ```python
  # 正しい:
  if not seq:
  if seq:
  # 間違い:
  if len(seq):
  if not len(seq):
  ```

- 行末空白に依存した文字列を使うな。分からんしエディタが勝手に消すぞ。

- `bool`に`==`使うな。

  ```python
  # 正しい:
  if greeting:
  # 間違い:
  if greeting == True:
  # もっと悪い:
  if greeting is True:
  ```

- `finally`で`return`/`break`/`continue`するな。`finally`から伝播する例外をもみ消すからな。

  ```python
  # 間違い:
  def foo():
      try:
          1 / 0
      finally:
          return 42
  ```

### 関数アノテーション

[PEP 484 – Type Hints](https://peps.python.org/pep-0484/)で関数アノテーションのルールも変わった。
- PEP484に従え(推奨事項(※スペースの入れ方とか)はこっちでも書いてある)。

- PEP484ではアノテーションの使い方を実験することが推奨されていたが、今は違う(※関数アノテーションはPEP3107で3.0から導入されていたが、その使途がPEP484で型ヒントに特化された)。
- 標準ライブラリ以外では、PEP484ルールの中で、型ヒント付けて分かりやすくなるか試したりできる。
- 標準ライブラリでアノテーションは控えめ(※conservative)にすべきだが、新しいコードや大規模リファクタリングではOK。
- 関数アノテーションをPEP484以外の意図で使うなら、ファイルの先頭あたりに
  ```python
  # type: ignore
  ```
  しておけば無視される。(PEP484に型チェックツールを黙らせる方法が書いてある)
- linterや型チェックツールはPythonインタプリタじゃない、使うかどうかも任意。アノテーションでPythonの挙動が変わることもない。
- 使いたくなければそうしろ。ただ他の人はパッケージを型チェックしたいと思うかもしれん、スタブファイル(`.pyi`)推奨。

### 変数アノテーション

[PEP 526 – Syntax for Variable Annotations](https://peps.python.org/pep-0526/)で変数アノテーションが導入された。推奨スタイルは関数アノテーションに似ている。
変数アノテーションは`:`の後に1つスペース。前に入れない、代入は`=`の前後スペース。
```python
# 正しい:
code: int
class Point:
    coords: Tuple[int, int]
    label: str = '<unknown>'

# 間違い:
code:int  # コロンの後にスペースがない
code : int  # コロンの前にスペース
class Test:
    result: int=0  # 等号の前後にスペースがない
```
PEP526は3.6～だが変数アノテーション文法は全バージョンのPythonスタブファイルで使われるべき。