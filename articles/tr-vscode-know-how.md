---
title: "Visual Studio Code キーボードショートカットとか(自分用メモ)"
emoji: "✍️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vscode"]
published: false
---
## 設定
`Ctrl`+`,`で設定画面
- `Enter`で勝手にサジェスチョンが選ばれないようにする
  Editor: Accept Suggestion On Enter

## キーボードショートカット

|キー|機能|
|--|--|
|`Alt+Z`|行末の折り返し切替|
|`Ctrl+P`|ファイル名を指定して開く|
|`Ctrl+X`|行の切り取り (未選択時)|
|`Ctrl+C`|行のコピー (未選択時)|
|`Alt+Down`|カーソル行を下に移動|
|`Alt+Up`|カーソル行を上に移動|
|`Shift+Alt+Down`|カーソル行を下にコピー|
|`Shift+Alt+Up`|カーソル行を上にコピー|
|`Ctrl+Shift+K`|カーソル行削除|
|`Ctrl+Enter`|下に行追加|
|`Ctrl+Shift+\`|次の対応する括弧に移動|
|`Ctrl+]`|行にインデントを追加|
|`Ctrl+[`|行のインデントを削除|
|`Ctrl+PgUp`|次のタブに移動|
|`Ctrl+PgDown`|前のタブに移動|
|`Ctrl+F`|検索|
|`Ctrl+H`|置換|
|`F3`|次を検索|
|`Shift+F3`|前を検索|
|`Ctrl+D`|次のマッチを選択に追加|
|`Ctrl+K Ctrl+D`|次のマッチに移動|
|`Ctrl+Alt+Down`|カーソルを下に追加|
|`Ctrl+Alt+Up`|カーソルを上に追加|
|`Ctrl+U`|カーソル動作のUndo|
|`Ctrl+L`|現在の行の選択|
|`Ctrl+Shift+L`|選択部分の全マッチを選択|
|`Shift+Alt+Right`|選択範囲を広げる|
|`Shift+Alt+Left`|選択範囲を縮める|
|`Ctrl+/`|行コメント記号をトグル|
|`Ctrl+W`|エディターを閉じる|
|`Ctrl+1`|左のエディターにフォーカス|
|`Ctrl+2`|サイドエディターにフォーカス|
|`Ctrl+3`|右のエディターにフォーカス|
|`Ctrl+N`|新しいファイル|
|`Ctrl+B`|サイドバー表示のトグル|
|`Ctrl+Shift+E`|エクスプローラーの表示|
|`Ctrl+Shift+G`|Git画面の表示|
|`Ctrl+Shift+F`|検索画面の表示|
|`Shift+Alt+I`|選択した行のすべての行末にカーソルを追加|


## 拡張機能

|名前|概要|用途|
|--|--|--|
|Japanese Language Pack for VS Code|名前の通り|まあ要るよね。|
|WSL|WSL上のdistroにアタッチしファイルを編集できる|**DockerやDev Containersあればあんまりいらないのでは？**|
|Markdown All in One|markdown(`.md`)記述用|GitHubからZennに投稿するのなんかに便利。ただコードブロックのシンタックスハイライト等微妙に違うので、Zenn CLIとの併用になる|
|Dev Containers| | |
|Docker| | |
|GitHub Repositories| | |
|GitHub Pull Requests| | |
