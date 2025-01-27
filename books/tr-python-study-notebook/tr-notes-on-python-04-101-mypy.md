---
title: "mypy"
---

Python3.5から導入された[「型ヒント(Type Hints)」](https://peps.python.org/pep-0484/)機構を使って、動的型付け言語のPythonに型チェックを行ってくれるもの。

## インストール

```shell-session
$ pip install mypy
```

## 使い方

### 基本的な使い方

```python
"""mypyの型チェックを確認するスクリプト"""

def hoge(piyo: int) -> str:
    return 'hoge' * piyo

res = hoge(3)
print(res)      #hogehogehoge
res = 1         #エラー! resはhoge()の戻り値でstrのため数値の代入は不可のはず
print(res)      #1
```

```shell-session
$ mypy mypy_test.py
mypy_test.py:8: error: Incompatible types in assignment (expression has type "int", variable has type "str")  [assignment]
Found 1 error in 1 file (checked 1 source file)
```

ちゃんと見つけてくれている。

```shell-session
$ # プロジェクト全体をチェック
$ mypy my_project/
```

引数にNoneが渡る可能性がある場合、`Optional`を使う。

```python
from typing import Optional

def hoge(piyo: Optional[int]) -> str:
```

ジェネリック型も可能。動的型付けのまま、実行時に引数と戻り値の型を同一に固定する。

```python
from typing import TypeVar, List

T = TypeVar('T')

def hoge(value: T, count: int) -> List[T]:
    return [value] * count

res = hoge('piyo', 3)
print(res)      # ['piyo', 'piyo', 'piyo']
res[0] = 1      # エラー! strの配列にintを入れようとしている
print(res)      # [1, 'piyo', 'piyo']
```

```shell-session
mypy_test1.py:10: error: No overload variant of "__setitem__" of "list" matches argument types "int", "int"  [call-overload]
mypy_test1.py:10: note: Possible overload variants:
mypy_test1.py:10: note:     def __setitem__(self, SupportsIndex, str, /) -> None
mypy_test1.py:10: note:     def __setitem__(self, slice[Any, Any, Any], Iterable[str], /) -> None
Found 1 error in 1 file (checked 1 source file)
root@b8b501c59a7e:/app#
```


## オプション

主なオプション:

|オプション|説明|
|--|--|
|`--ignore-missing-imports`|外部ライブラリの型チェックを無視。通常のプロジェクトで便利|
|`--strict`|より厳格な型チェック|
|`--check-untyped-defs`|型ヒントがない関数もチェック(デフォルトでは無視)|
|`--no-strict-optional`|オプショナル型(Optional)に関連する厳格なチェックを無効化|
