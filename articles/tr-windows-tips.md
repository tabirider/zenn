---
title: "Windowsのストレスを軽減する細かいtips"
emoji: "✍️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["windows"]
published: true
---

随時更新。

## エクスプローラを何とかしたい

たかがファイラが何でこんな重くて使いにくいんだよ。
フォルダツリーの階層を示す点線が出なくなったから、複雑なフォルダ構造になるとどれとどれが同じ階層なのか、画面に定規あてないと分からないレベル。
![hoge](/images/tr-windows-tips/explorer.png)
ついでに大量の画像ファイルを扱おうとすると勝手に画像プレビューしようとするから、あり得ないくらい重くなる。
[秀丸ファイラーClassic](https://hide.maruo.co.jp/software/hmfilerclassic.html)が古のスタイルを呼び戻してくれる。昭和生まれのオッサン必需品、お金を払う価値がある。
実家のような安心感。
![](/images/tr-windows-tips/hidemaru-filer.png)
これだけ違う。
![](/images/tr-windows-tips/explorer-vs-hidemaru-filer.png)

## 画像関連

秀丸ファイラは画像プレビュー機能ないので[XnView](https://www.xnview.com/en/)併用。これも昭和感漂うけど、切り抜き・リサイズ・簡単な補正くらいには十分な機能。
![](/images/tr-windows-tips/xn-view.png)

もうちょい本格的にいじるなら[GIMP](https://www.gimp.org/)があります。
![](/images/tr-windows-tips/gimp.png)

~~ペイントにImageCreatorつけたりする前に、やることあるんじゃないですかねぇMSさん。~~

## スタートメニューがイミフwww

バカでかい面積でこんだけしか見えなくて延々スクロールさせるのやめてもらっていいですか？

![](/images/tr-windows-tips/windows-start-menu.png)

はいはい[秀丸スタートメニュー](https://hide.maruo.co.jp/software/hmstartmenu.html)。昭和生まれ(略
![](/images/tr-windows-tips/hidemaru-start-menu.png)

## アプリ起動にマウス使いたくない

ランチャーで使いやすいのは定番[CLaunch](https://forest.watch.impress.co.jp/library/software/claunch/)。ホットキー`Ctrl`+`Shift`+`Z`で上げてアプリにもショートカットキー割り当てればマウスから開放される。
![](/images/tr-windows-tips/claunch.png)

ただ、**マイクロソフトストア**とかいうところからインストールしたソフトは、このランチャーにドラッグ&ドロップできない(CLaunchのせいじゃなくて、ストアアプリ共通の性質)。ファイルパスをベタ書きで登録しようにも、**まともな手段でそのアプリのパスを知る方法さえない**。
アプリを起動してから`Ctrl`+`Alt`+`Delete`でタスクマネージャを開いてアプリを探す。右クリックでプロパティ。
![](/images/tr-windows-tips/claunch-find-path-1.png)
**説明**に**ファイル名**が、**場所**に**ファイルパス**がある。コピーできなさそうだけどできる。
![](/images/tr-windows-tips/claunch-find-path-2.png)
これを組合せてCLaunchに登録。
![](/images/tr-windows-tips/claunch-regist-app.png)
しかも、**アップデートのたびにパスが変わるのでリンク切れたらこの手順でやり直し**。ほんと何をどうしたら~~くｓぇれｗｆｄさｋｌｒ~~

## PowerToysが良さそう

[Windows Power Toys](https://learn.microsoft.com/en-us/windows/powertoys/)(25年1月時点でプレビュー版)。いいじゃん。もう「環境変数のアレってどこからどう辿るんだっけ？」と悩まなくていい。
![](/images/tr-windows-tips/ms-power-toys.png)

### Windowsを勝手にスリープさせない

~~訳の分からない~~再起動をこれで防げるかは未検証。

![](/images/tr-windows-tips/ms-power-toys-awake.png)

### 大きいディスプレイで自由にウインドウ分割

WQHDや4Kの30型以上を使ってる人には便利そう。

![](/images/tr-windows-tips/ms-power-toys-fancy-zones.png)

### Shiftキーなしでアンスコ入力

JISキーボードでは`BackSpace`キー左の「￥」キーと、右`Shift`キー左の`\`キーは同じASCIIコード「￥」(5C)を送出する。つまり「￥」の入力に右上キーを使えば、右下の`\`キーは不要。そして、`_`は`Shift`+`\`に割り当てられている。`snake_case`系言語では圧倒的に`_`タイプが多いので、`Shift`なしで`_`入力できれば効率が上がる。
同じようなツール他にもあるけど、純正でできるようになりました。
`Keyboard Manager`で「キーの再マップ」→「選択」→右`Shift`の左のキー(`BackSpace`横の「\」キーでも同じ表示になるけど、ちゃんと識別されてる)

![](/images/tr-windows-tips/ms-power-toys-keyboard-manager-1.png)

アンスコに変更。これで`snake_case`系言語の効率2%アップ

![](/images/tr-windows-tips/ms-power-toys-keyboard-manager-2.png)

他にも使い勝手よさそうな機能があるので、暇を見て試してみる。

## 変換・無変換で日本語入力ON/OFF

`Alt`+`半角・全角`はすごく押しにくい。特に技術系のメモ取ったりマニュアル作るときは半角全角が入り混じって気が狂いそうになる。変換はスペースキーで間に合うので、すごくいい位置を占領してる`変換`と`無変換`をIME切替に割り当てれば、親指で押せて快適。

### Google日本語入力の場合

![](/images/tr-windows-tips/google-ime-key-mapping-1.png)
![](/images/tr-windows-tips/google-ime-key-mapping-2.png)

### Microsoft IMEの場合

![](/images/tr-windows-tips/ms-ime-key-mapping.png)

## フォントのインストール・管理

`スタート`→`フォント`とタイプ→`フォントの設定`
または好きなフォントの`OTF`ファイルをダウンロードして右クリック
インストールされているフォントを見ることはできるけど、**悲しいくらい見にくい**。おまけに肝心の**フォント名をコピーする機能がどこにも見当たらない**。コンパネからエクスプローラで開くレガシーのフォント管理でもそう、ちょっと訳が分からない。何かしようとすると、毎回Windowsは「なんでこれができないの！？」が多すぎる。
仕方ないので[nexusfont](https://www.xiles.app/)導入。見やすくて安心する。
![](/images/tr-windows-tips/nexusfont.png)
