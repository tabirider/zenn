---
title: "Zenn-snippet"
emoji: "✍️"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["zenn"]
published: false
---
<!-- 見出し -->
## 見出し
### 見出し
#### 見出し
<!-- リスト -->
- リスト
- リスト
  - リスト(ネスト)
  * リスト
1.  番号付きリスト
2.  番号付きリスト
    1. 番号付きリスト(ネスト)
        リストの中の記事

<!-- リンク -->
[アンカーテキスト](https://www.google.co.jp/)

<!-- 画像 -->
![](/images/zenn-snippet/sample.png)
[![](/images/zenn-snippet/linked-image.png)](https://www.google.co.jp/)

<!-- テーブル -->
|head|head|
|--|--|
|text|text|

<!-- コードブロック -->
```python
print(id(hoge))
```

```python:hoge.py
print('hoge')
```

```
コードブロックに指定できるハイライト:
https://prismjs.com/#supported-languages
html, css, js, c, cpp, csv, diff, docker, git, gitignore, go, http,
java, json, json5, jsonp, kotlin, log, md or markdown, mermaid, mongodb,
nginx, prel, php, plsql, powershell, py, shell-session, sql, tcl,
typescript or ts, url, wiki, yaml, etc.
```

```diff js
@@ -4,6 +4,5 @@
    const foo = bar.baz([1, 2, 3]) + 1;
    let foo = bar.baz([1, 2, 3]);
```

<!-- 数式 -->
[KaTeX](https://katex.org/docs/support_table.html)

<!-- 数式ブロック -->

$$
e^{i\theta} = \cos\theta + i\sin\theta
$$

インラインの$a\ne0$数式埋め込み

<!-- 引用 -->
> 引用文

<!-- 脚注 -->
脚注[^1]の例。
インライン^[インラインの脚注の内容]も可能。

[^1]: 脚注の内容

<!-- 区切り線 -->
---

インライン*italic*、**太字**, ~~取り消し線~~, `code`

<!-- Zenn独自拡張 -->
::::message
メッセージ
::::

:::message alert
警告
:::

:::details アコーディオン
その中身
1.  番号付き
    ```
    コードブロック
    ```
:::

:::::details いろいろネスト
  ::::message
  メッセージ
  :::message alert
  さらにネスト
  :::
  ::::
:::::

<!-- カード -->
https://www.yahoo.co.jp
<!-- カードがうまくいかない場合 -->
@[card](https://www.yahoo.co.jp)
<!-- リンクのみ -->
<https://www.yahoo.co.jp>

<!-- YouTube -->

https://www.youtube.com/watch?v=x3qaFHhh1jQ

<!-- GitHub -->
[github-sample/sample.txt](https://github.com/tabirider/zenn/blob/d45918a9ab42869fee0da3f2ca283e952e93dafc/github-sample/sample.txt)

https://github.com/tabirider/zenn/blob/d45918a9ab42869fee0da3f2ca283e952e93dafc/github-sample/sample.txt

https://github.com/tabirider/zenn/blob/d45918a9ab42869fee0da3f2ca283e952e93dafc/github-sample/sample.txt#L3-L5