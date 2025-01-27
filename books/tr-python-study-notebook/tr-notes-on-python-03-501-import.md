---
title: "インポート"
---

## importの動作

```python
import module[ as identifier][, module[ as identifier], ..]
# →名前空間はmoduleで分離される。identifierで別名をつけられる
import module
# →module.func_name()
import module as md
# →md.func_name()
from module import *
# →module内の名前空間をローカル名前空間に引き込む(非推奨)
from module import func_name1, func_name2, ..
# →特定の関数をローカル名前空間に引き込む
from module import func_name as identifier
# →特定の関数を別名で使う
from module as identifier import *  # Ｘ
# →moduleをローカル名前空間に引き込むのでas指定の意味がない
```
:::message alert
`from`の場合、サブモジュールを辿る処理が入る。このへんはあとで
<https://docs.python.org/ja/3.13/reference/simple_stmts.html#index-34>
:::
`importlib.import_module()`で動的にロードできる

## パッケージ

通常のパッケージ、名前空間パッケージがある。

### 通常のパッケージ

`__init__.py`を含むディレクトリ。`__init__.py`はパッケージの`import`で読み込まれ、そこで定義されるオブジェクトがパッケージの名前空間に入る。
`3.3`以降は`__init__.py`がなくてもディレクトリがパッケージと認識されるようになった(がパッケージと明示するために空でも作るのが一般的)。パッケージを初期化したければ必要。
`__init__.py`で`__all__`を定義することで、公開モジュールや名前を制御できる。

```
mypackage
+ __init__.py
+ module1.py
+ module2.py
+ sub_package
+ + __init__.py
+ + module3.py
```

```python:mypackage/__init__.py
from .module1 import hoge_func
from .module2 import piyo_func
```

```python:module_user.py
import mypackage
```

### __all__

公開範囲を設定する。
```python
__all__ = ["func1", "func2"]
```
`__all__`に含まれないオブジェクトは`import`しても外部に公開されない。
通常は`_func()`や`__func()`で十分。大規模プロジェクトでパッケージの公開APIを明確に定めたいときに有用。`__init__.py`の中で必要なモジュールを`imnport`し、`__all__`で公開範囲を設定する。
